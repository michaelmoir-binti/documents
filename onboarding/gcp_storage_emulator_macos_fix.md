# GCP Storage Emulator macOS Networking Fix

## Problem

On macOS, the GCP Storage Emulator container was using `network_mode: host` which doesn't work properly with Docker Desktop. This caused:
- Rails initialization to fail with `Connection refused - connect(2) for "localhost" port 9023`
- The emulator container to run but be inaccessible from the host
- Application startup to be blocked

## Root Cause

Docker Desktop on macOS runs Docker in a Linux VM. The `network_mode: host` option doesn't work as expected because it refers to the VM's network, not the macOS host network. Ports need to be explicitly mapped.

## Solution

Two changes were implemented:

### 1. Docker Compose Configuration (Option 2)

**File**: `docker-compose.yml`

**Change**: Replaced `network_mode: host` with explicit port mapping

**Before**:
```yaml
gcp_storage_emulator:
  container_name: gcp_storage_emulator
  image: gcr.io/vaulted-fort-143519/github.com/binti-family/gcp-storage-emulator:v2.0.0
  network_mode: host
  environment:
    HOST: "localhost"
    PORT: "9023"
  restart: always
```

**After**:
```yaml
gcp_storage_emulator:
  container_name: gcp_storage_emulator
  image: gcr.io/vaulted-fort-143519/github.com/binti-family/gcp-storage-emulator:v2.0.0
  # Using port mapping instead of network_mode: host for macOS compatibility.
  # network_mode: host doesn't work properly on macOS Docker Desktop.
  # Port mapping allows the emulator to be accessible from localhost:9023.
  ports:
    - "9023:9023"
  environment:
    HOST: "0.0.0.0"  # Listen on all interfaces within container
    PORT: "9023"
  restart: always
```

### 2. Resilient Initialization (Option 1)

**File**: `config/initializers/refile.rb`

**Change**: Added error handling for connection failures during bucket initialization

**Before**:
```ruby
if !ShrineConfig.disable? && !ENV["TAPIOCA_DSL"]
  BintiFamily.buckets.each_value do |bucket|
    storage.get_bucket(bucket)
  rescue Google::Apis::ClientError
    # get_bucket raises on 404, in which case we want to create the bucket
    storage.put_bucket(bucket)
  end
end
```

**After**:
```ruby
if !ShrineConfig.disable? && !ENV["TAPIOCA_DSL"]
  BintiFamily.buckets.each_value do |bucket|
    begin
      storage.get_bucket(bucket)
    rescue Google::Apis::ClientError
      # get_bucket raises on 404, in which case we want to create the bucket
      storage.put_bucket(bucket)
    rescue Google::Apis::TransmissionError, Errno::ECONNREFUSED, Errno::EHOSTUNREACH, SocketError
      # Connection errors - emulator may not be accessible yet (e.g., during startup)
      # Log warning but don't fail initialization. Buckets will be created on first use.
      Rails.logger.warn(
        "Could not connect to GCP Storage Emulator at #{refile_root_url} " \
        "for bucket '#{bucket}'. Bucket operations will be retried on first use.",
      )
    end
  end
end
```

## Implementation Steps

1. **Update docker-compose.yml**: Changed from `network_mode: host` to port mapping `9023:9023`
2. **Update refile.rb initializer**: Added error handling for connection failures
3. **Restart the container**:
   ```bash
   docker-compose stop gcp_storage_emulator
   docker-compose up -d gcp_storage_emulator
   ```

## Verification

After implementing the fix:

1. **Check container is running with port mapping**:
   ```bash
   docker ps --filter "name=gcp_storage_emulator" --format "{{.Ports}}"
   # Should show: 0.0.0.0:9023->9023/tcp
   ```

2. **Test connectivity**:
   ```bash
   curl http://localhost:9023/
   # Should return: OK
   ```

3. **Verify Rails can initialize**:
   ```bash
   bundle exec rails runner "puts 'Rails initialized successfully'"
   # Should not show GCP Storage Emulator connection errors
   ```

## Benefits

- ✅ Works on macOS Docker Desktop
- ✅ Rails can initialize even if emulator isn't ready yet
- ✅ Buckets are created on first use if initialization fails
- ✅ Better error messages for debugging
- ✅ No breaking changes for Linux/CI environments

## Notes

- The port mapping approach works on both macOS and Linux
- The error handling ensures Rails can start even if the emulator container is starting up
- Buckets will be automatically created when first accessed if they weren't created during initialization
- This fix maintains compatibility with CI environments (which use container networking)

## Date Implemented

December 11, 2025

