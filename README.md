# Bug-Bounty-Reports
Bug bounty findings, vulnerability research, and responsible disclosure reports.


Summary

The Promo Code API endpoint exposes sensitive internal implementation details when processing invalid user input.

Instead of returning a generic validation error, the application reveals backend technology information, internal service names, class names, and validation rules within the response body.

Additionally, the endpoint returns an HTTP 500 Internal Server Error for expected input validation failures, indicating improper exception handling.

This information disclosure may assist attackers during reconnaissance activities and facilitate more targeted attacks against the application infrastructure.

Vulnerability Details

Endpoint: /api/PromoCode

Parameter: promoCode

The endpoint performs server-side validation on the supplied promo code value.

When an invalid value exceeding the allowed length is submitted, the backend returns a verbose exception message containing internal implementation details.

The response reveals:

Backend platform information.

Internal service names.

Internal class names.

Promo code length restrictions.

Validation implementation details.

Furthermore, the application returns HTTP 500 Internal Server Error instead of an appropriate client-side validation response, such as HTTP 400 Bad Request.

Proof of Concept (PoC)

Step 1 – Intercept the Request

Intercept a request to the Promo Code API using Burp Suite and send it to Repeater.

Step 2 – Submit Invalid Input

Send the following request with a promo code exceeding the allowed length:

POST /api/PromoCode HTTP/1.1
Host: [REDACTED]
Content-Type: application/json

{
  "promoCode": {
    "action": "add",
    "promoCode": "22222222+"
  }
}

Step 3 – Observe the Response

The application returns an HTTP 500 response containing internal implementation details:

HTTP/1.1 500 Internal Server Error

{
  "errors": [
    {
      "errorMessage": "Validation Failed on call of [Internal Service] ApplyPromotionRequest.PromotionCode : The value of ApplyPromotionRequest.PromotionCode: LengthStringAttribute is '22222222+' but must be no more than 8 characters in length.",
      "errorCode": "",
      "errorType": "",
      "severity": "Error"
    }
  ]
}

Step 4 – Analyze the Response

The response reveals several internal details:

HTTP 500 is returned instead of an appropriate validation status code.

Internal service and class names are exposed.

Backend implementation details are disclosed.

Input validation constraints are revealed.

Requirements

No authentication required.

No special privileges required.

Reproducible with invalid input.

Impact

1. Technology Stack Disclosure

The response exposes information about the underlying backend platform and framework components.

An attacker may use this information to identify publicly known vulnerabilities, security advisories, or platform-specific attack vectors.

2. Internal Architecture Disclosure

The application reveals internal implementation details, including:

Service interfaces.

Request object names.

Validation components.

This information may assist attackers in mapping the application's internal structure and understanding backend workflows.

3. Business Logic Disclosure

The error message reveals validation rules associated with promo code processing, including maximum accepted length restrictions.

This information may help attackers understand application behavior and optimize further testing activities.

4. Unauthenticated Information Disclosure

The issue can be triggered without authentication or special privileges, increasing the application's exposure surface and making automated reconnaissance easier.

Risk Rating

Severity: Informational

Although the vulnerability does not directly allow unauthorized access or data modification, it exposes internal application details that may facilitate further attacks and reduce uncertainty during reconnaissance.

Remediation

The following security improvements are recommended:

Replace verbose exception messages with generic user-facing error responses.

Avoid exposing internal class names, namespaces, method names, framework identifiers, or validation attributes in API responses.

Return appropriate HTTP status codes for validation failures, such as:

HTTP 400 Bad Request.

HTTP 422 Unprocessable Entity.

Implement centralized exception handling to ensure unexpected errors are logged internally while returning sanitized responses to clients.

Review all API endpoints for similar verbose error disclosures and standardize secure error handling practices.

Tools Used

Burp Suite Repeater

Manual API Testing

HTTP Request and Response Analysis

Disclosure

The vulnerability was identified and responsibly reported to the affected organization.

Sensitive information, internal implementation details, and identifying information have been redacted from this public report.

