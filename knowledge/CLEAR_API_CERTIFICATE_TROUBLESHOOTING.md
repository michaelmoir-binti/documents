# CLEAR API Certificate Troubleshooting - Contact Information for CLEAR Support

**Date:** December 29, 2025  
**Issue:** 401 Authentication Error (sub_status 10003) when calling CLEAR API `/v3/person/searchResults` endpoint  
**Environment:** Beta (`s2s.beta.thomsonreuters.com/api`)

## Summary

We are experiencing a 401 Unauthorized error (sub_status 10003) when attempting to make API calls to the CLEAR Person Search endpoint. Our investigation confirms that:

- ✅ SSL connection is working correctly
- ✅ Client certificate is being sent in the SSL handshake
- ✅ Basic Authentication credentials are configured
- ✅ Certificate and key files are valid and properly formatted
- ❌ API endpoint `/v3/person/searchResults` returns 401, suggesting certificate may not be enabled/activated on CLEAR's server

## Certificate Details

**Certificate Subject:**
```
CN=Binti Inc Beta
O=Binti Inc
OU=Binti Inc
L=Oakland
ST=Ca
C=US
emailAddress=jamie@binti.com
```

**Certificate Issuer:**
```
C=US
O=Thomson Reuters Holdings Inc.
OU=Thomson Reuters Hosted Services PKI
CN=Thomson Reuters CLEAR S2S Beta Issuing CA1
```

**Certificate Serial Number:**
```
694344509614741081943121995306882535669703342120
```

**Certificate SHA256 Fingerprint:**
```
47:E4:04:5C:A8:5F:19:D8:1F:F4:EC:35:FF:0E:0F:C2:9E:82:21:D6:E7:B4:21:DA:63:EA:BD:F4:4B:B3:3E:39
```

**Certificate Validity:**
- Not Before: Apr 16 00:00:00 2025 GMT
- Not After: Apr 16 23:59:59 2030 GMT

**Certificate Authority Information:**
- OCSP URI: `http://ocsp.one.digicert.com`
- CA Issuers URI: `http://cacerts.one.digicert.com/ThomsonReutersCLEARS2SBetaIssuingCA1.crt`
- CRL Distribution Point: `http://crl.one.digicert.com/ThomsonReutersCLEARS2SBetaIssuingCA1.crl`

## Technical Verification

### What We've Verified

1. **Certificate Files:**
   - Certificate file: `Binti_CLEAR_S2S_Beta.certificate.pem` (2,004 bytes)
   - Key file: `Binti_CLEAR_S2S_Beta.key.pem` (1,679 bytes)
   - Both files are readable and properly formatted PEM files
   - Certificate and key match (same modulus verified)

2. **SSL Connection:**
   - Direct SSL connection to `s2s.beta.thomsonreuters.com:443` succeeds
   - Certificate is being presented during SSL handshake
   - Server accepts the SSL connection

3. **Authentication:**
   - Basic Auth credentials are configured (username and password set)
   - GET request to root path `/` returns 404 (not 401), confirming:
     - SSL handshake succeeds
     - Certificate is accepted
     - Basic Auth is working

4. **Code Configuration:**
   - Certificate paths are correctly configured
   - SSL options are properly set in Faraday client
   - Client certificate and key are loaded correctly

### Error Details

**API Endpoint:** `POST https://s2s.beta.thomsonreuters.com/api/v3/person/searchResults`

**Response:**
- Status: 401 Unauthorized
- Sub Status: 10003
- Error occurs specifically on this endpoint, not on root path

**Request Details:**
- Method: POST
- Content-Type: `application/xml`
- Accept: `application/xml`
- Authentication: HTTP Basic Auth (username/password)
- SSL: Client certificate authentication (mutual TLS)

## Questions for CLEAR Support

1. **Certificate Activation:**
   - Is the certificate with serial number `694344509614741081943121995306882535669703342120` currently enabled/activated for our account?
   - Does the certificate need to be explicitly activated for the Beta environment?

2. **Endpoint Authorization:**
   - Is the certificate authorized to access the `/v3/person/searchResults` endpoint?
   - Are there any account-level restrictions that might prevent API access?

3. **Certificate Status:**
   - What is the current status of this certificate in your system?
   - Are there any pending approvals or activation steps required?

4. **Account Verification:**
   - Can you confirm the certificate is associated with the correct account/username?
   - Username: `4275229` (from our configuration)

5. **Error Code 10003:**
   - What does sub_status code 10003 specifically indicate?
   - Is this related to certificate authentication or another authentication issue?

## Configuration Details

**Environment:** Beta  
**Base URL:** `https://s2s.beta.thomsonreuters.com/api`  
**Authentication Method:** 
- SSL Client Certificate (mutual TLS)
- HTTP Basic Auth (username/password)

**SSL Configuration:**
- Client certificate: Present and valid
- Certificate verification: Enabled (VERIFY_PEER)
- SSL connection: Successful

## Test Results

### Successful Tests:
- ✅ SSL handshake with client certificate
- ✅ Connection to root path (returns 404, not 401)
- ✅ Certificate and key file validation
- ✅ Certificate format and validity checks

### Failing Tests:
- ❌ POST to `/v3/person/searchResults` returns 401 (sub_status 10003)

## Next Steps

1. Contact CLEAR support with this document
2. Provide certificate serial number and fingerprint
3. Request verification of certificate activation status
4. Request confirmation of endpoint authorization
5. Ask for clarification on error code 10003

## Contact Information

**Account:** Binti Inc Beta  
**Certificate Contact Email:** jamie@binti.com (from certificate)  
**Certificate Subject:** Binti Inc Beta

---

**Note:** This document was generated after comprehensive troubleshooting. All code-level configuration has been verified as correct. The issue appears to be server-side certificate activation/authorization.

