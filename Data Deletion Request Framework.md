
<h1>Data Deletion Request Framework</h1>

<h2 id="about-this-document">About this document</h2>
<p>The Data Deletion Request Framework provides a standard approach for handling data deletion requests within the adtech ecosystem. This framework establishes a standardized mechanism for transmitting data deletion requests signals throughout the digital advertising chain. It includes provisions for recipients to indicate necessary requirements for executing deletion requests, validate request origins, ensure requester authenticity, confirm receipt, and employ cryptographic signatures for request and receipt authentication.</p>

<h3>Version History&nbsp;</h3>
<div>
<table>
<tbody>
<tr>
<td><strong>Date</strong></td>
<td><strong>Version</strong></td>
<td><strong>Comments</strong></td>
</tr>
<tr>
<td>April 2026</td>
<td>2.0</td>
<td>Version 2.0 released</td>
</tr>	
<tr>
<td>May 2024</td>
<td>1.0</td>
<td>Version 1.0 released</td>
</tr>
</tbody>
</table>
</div>

<h2>Introduction</h2>
<p>The ‘Right to Delete’ is a Data Subject Right (DSR) included as part of the GDPR, US state privacy laws, and additional privacy legislation such as Quebec Law 25.&nbsp;</p>
<p>The US Privacy
  <a target="_blank" href="https://github.com/InteractiveAdvertisingBureau/USPrivacy/blob/master/CCPA/Data%20Deletion%20Request%20Handling.md">
    <span style="color:rgb(17, 85, 204);">specification</span>
  </a>, which was deprecated on 31 January 2024, supported California Consumer Privacy Act (CCPA) requirements with an included data deletion request handling mechanism. However, the original mechanism only supported the CCPA, which is now too limited a scope. As privacy legislation evolves across California, other US state laws, and global jurisdictions beyond GDPR, the data deletion request framework needs to also evolve.&nbsp;</p>
<p>Handling data deletion request signals is a challenge for the adtech ecosystem. Existing DSR tools in-market do not have integrations with adtech companies. To support adtech, we need a protocol that successfully communicates data deletion requests to participants, ensuring all related parties who share data can receive the request signal. This specification describes a data deletion signaling mechanism that is intended to be an interoperable standard within the Global Privacy Platform (GPP).</p>
<h4>Terms</h4>
<p>The following terms have specific meaning within this document:</p>
<ul>
  <li>Deletion Request</li>
  <ul>
    <li>A deletion request is a request made by a user to a party, referred to as a 1st-party, which they have a direct relationship with.</li>
  </ul>
  <li>Deletion Request Transaction</li>
  <ul>
    <li>A deletion request transaction is an interaction consisting of the communication of a deletion request by a requester and the acknowledgement of the request by a recipient.</li>
  </ul>
  <li>Deletion Request Chain</li>
  <ul>
    <li>A deletion request chain is a series of one more deletion request transactions originating from a given requester.</li>
  </ul>
  <li>1st Party</li>
  <ul>
    <li>A 1st Party (also first-party) is an adtech ecosystem participant which has direct relationships with users and which may receive requests from those users to delete their data.</li>
  </ul>
  <li>Vendor</li>
  <ul>
    <li>A Vendor is an adtech ecosystem participant which has direct or indirect relationships with 1st Parties and which may receive deletion requests from those 1st Parties.</li>
  </ul>
  <li>Requester</li>
  <ul>
    <li>A requester is a party which initiates a data-deletion transaction by communicating a data deletion request on behalf of a user. A 1st-party is always the first requester in a deletion request chain.</li>
  </ul>
  <li>Recipient</li>
  <ul>
    <li>A recipient is a party which receives a deletion request from a requester. Recipients may themselves become requesters in succeeding transactions.</li>
  </ul>
</ul>
<h2>Requirements</h2>
<ol>
  <li>Provide a standard protocol that allows parties to communicate data deletion request signals.</li>
  <li>Provide a standard for recipients to indicate what must be provided and how it must be packaged to execute data deletion requests.</li>
  <li>Provide a standard for recipients of data deletion requests to validate the 1st-party origin of a request.</li>
  <li>Provide a standard means by which recipients of data deletion requests can determine the authenticity of the requester.</li>
  <li>Provide a standard for recipients to confirm receipt of a data deletion request.</li>
  <li>Provide a standard for parties to have deletion requests and request receipts cryptographically signed by those making and receiving the requests, respectively.</li>
</ol>
<h4>Scope</h4>
<p>This proposal is focused specifically on defining a two stage process by which deletions are requested and affirmed. In the first stage, requests are communicated from a requester to a recipient and the recipient acknowledges receipt of the request. A recipient may also become a requester if they need to forward the request to partners.</p>
<h4>Out-of-scope</h4>
<ol>
  <li>Verifying the original data-subject request: it is assumed that first-parties will be responsible for authenticating and validating requests prior to communicating them via this protocol.&nbsp;&nbsp;</li>
  <li>How deletion request recipients act on them: it is assumed each participant will decide independently what their obligations for acting on a request and what their requirements are for satisfying those obligations.&nbsp;</li>
  <li>Supporting any other types of data subject rights requests.</li>
  <li>Determining what partners a participant must communicate deletion requests to: it is assumed participants will be responsible for deciding which of their partners they must forward deletion requests to in order to properly honor them.</li>
</ol>
<h4>Resources Required</h4>
<p>All participants will be required to publish a file with configuration parameters needed by other participants who support the standard. The file must be called dsrdelete.json and available at a standard location that it is easily discoverable by other participants.</p>
<p>The following information is required from participants in their dsrdelete.json resource:</p>
<ul>
  <li>All participants must publish cryptographic public keys conforming to the JSON Web Key (JWK) standard (<a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7517">
      <span style="color:rgb(17, 85, 204);">https://datatracker.ietf.org/doc/html/rfc7517</span>
    </a>), which will be used by other participants to verify JWT signatures generated with the associated private keys.</li>
  <li>All participants must publish in their dsrdelete.json files information 1st Parties will use to determine how to submit deletion requests to them and what the requirements for submitting the requests are (e.g. what identifiers/data must be provided and what API endpoint deletion requests must be sent to).</li>
</ul>
<ul>
  <li>All participants must publish an API endpoint as defined in their dsrdelete.json to which requesters can send deletion requests.</li>
</ul>
<h4>JSON Web Token (JWT) Implementation</h4>
<p>This specification employs the JSON Web Tokens (JWT) standard, which supports&nbsp; verifiable transmission of data through the use of cryptographic signatures, to assure deletion requests are valid and authentic. Requesters sign JWTs using their private keys to ensure provenance and authenticity, while recipients verify them using public keys hosted on the requester's domain.</p>
<p>Additionally, this specification leverages some of the registered public claims defined in the JWT standard to take advantage of the existing, generally accepted data value definitions and&nbsp; provide consistency across implementations. It also introduces custom private claims designed specifically for the deletion request use case.</p>
<p>The specification delineates three distinct JWTs: an originating requester JWT (orJWT), a request JWT (rqJWT), and an acknowledgment JWT (acJWT). Each serves a unique purpose as described below:</p>
<ol>
  <li>
    <strong>Originating Requester JWT (orJWT)</strong>: the orJWT is generated by the 1st party and includes information used to identify the origin of the deletion request. It's purpose is to provide, through a cryptograph signature, a verifiable proof that the request was generated by the originating party and hasn't been tampered with.</li>
  <li>
    <strong>Request JWT (rqJWT):</strong>
    The rqJWT includes the orJWT as a claim along with the identifier to which the deletion request applies and additional information needed to validate, authenticate and track the request. The originating 1st party will generate an rqJWT for each vendor they communicate a deletion request to. If vendors need to communicate the requests to partners they work with, they will generate an new rqJWT for each partner which includes a copy of the orJWT they received along with their identifier for the deletion request, which may be different from the identifier they received.&nbsp;</li>
  <li>
    <strong>Acknowledgement JWT (acJWT):</strong>
    Generated by a recipient and returned to a requester. The acJWT includes the rqJWT alongside acknowledgement information, including success or an error status.</li>
</ol>
<p>This model ensures all recipients are able validate and authenticate that the request originated with the claimed 1st party by verifying the orJWT signature using public keys retrieved from the originating domain. In addition, all recipients are able to verify the requests they receive originated with the claimed requester by verifying the rqJWT signature using public keys retrieved from the requester domain. Lastly, recipients are able to provide requesters with proof of the deletion request transaction and the success or failure of its communication with the acJWT signature that can be verified using public keys retrieved from the recipient domain.</p>
<p>For detailed information on JWT implementation, please refer to:
  <a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519">
    <span style="color:rgb(17, 85, 204);">IETF RFC 7519 - JSON Web Tokens (JWT)</span>
  </a>.</p>
<h2>Deletion Request Sequence</h2>
<h4>Request Sequence</h4>
<p>Deletion request sequences always begin with a user requesting that a 1st Party delete their data through a flow provided by the 1st Party as identified in step 1 below. Once the 1st Party has determined that the request needs to be communicated to partners, which partners it must be communicated to, and what information they must be provided with – a series of one or more discrete transactions occurs to propagate the request for each individual identifier subject to the request to all concerned parties.&nbsp;</p>
<p>The two parties in these transactions are a
  <strong>requester</strong>, who has received a deletion request from a data subject, and a
  <strong>recipient</strong>, who the requester has a data relationship with. The interactions between the requester and recipient are described in steps 2 through 10 below. If a recipient has additional partners with whom they have shared the data to be deleted, they will in turn become the requester in subsequent transactions with their partners.</p>
<p>A given deletion request may be forwarded by a requester to one or more than one recipient and the recipient may in turn forward the request to one or more subsequent recipients. The result will be chains of one or more transactions which fan out across the partner graph.</p>

<ol>
  <li><span style="color:rgb(255, 0, 0);">
    <strong>*Data Subject</strong> requests a data deletion from a 1st Party*</span></li>
  <ol>
    <li>1st Party validates the request.</li>
    <li>1st Party determines what identifiers are subject to the request.</li>
    <li>1st Party determines partners that data has been shared with which the request must be communicated to.</li>
    <li>1st party accesses the dsrdelete.json resource of each identified partner to determine how IDs must be formatted for them.&nbsp;</li>
    <li>1st party generates an orJWT for the request.</li>
  </ol>
  
  <li>
    <strong>Requester</strong> generates a deletion packet formatted as a rqJWT which includes the orJWT created in step 1.5, as well as the other values described under <a href="#request-data">Request Data</a>
    below.</li>
  <li>
    <strong>Requester</strong> sends the rqJWT to the Recipient.</li>
  <li>
    <strong>Recipient</strong> receives the rqJWT and verifies the signatures of the orJWT and rqJWT using public keys published on the 1st party and requester’s respective domains.&nbsp;</li>
  <li>
    <strong>Recipient</strong> acknowledges receipt to the requester with an acJWT. The acJWT includes the original rqJWT and additional acknowledgement values described under Recipient Acknowledgement, including a result code indicating successful receipt of the request or an error.</li>
  <li>
    <strong>Requester</strong> verifies the signatures of the acJWT and the embedded rqJWT returned by the Recipient using a public key published on the Recipient’s domain and their own public key.</li>
  <li><span style="color:rgb(255, 0, 0);">
    <strong>*Requester</strong> logs the Recipient Acknowledgement acJWT.*</span></li>
  <li>
    <span style="color:rgb(255, 0, 0);"><strong>*Recipient</strong>
    logs the request rqJWT.*</span></li>
  <li>
    <span style="color:rgb(255, 0, 0);"><strong>*Recipient</strong>
    forwards the request as necessary by:</span></li>
  <ol>
    <li>Determining what partners the request must be forwarded to.</li>
    <li>Accessing the dsrdelete.json resource for each partner to determine how IDs must be formatted for them.&nbsp;</li>
    <li>Generating rqJWTs to be sent to partners using the original orJWT they received and by following steps 2 through 8 above.&nbsp;&nbsp;</li>
  </ol>
  <li>
    <strong>*Recipient</strong>, in a separate flow, executes the data deletion.*</li>
</ol>
<h4>Out-of-scope</h4>
<p>The following steps have been marked with an asterisk because they are not part of the standard:</p>
<ul>
  <li>Step 1 and Steps 7-10.</li>
</ul>
<h4>JSON Web Token (JWT) Propagation</h4>
<p>When a deletion request needs to be propagated to downstream recipients, orJWTs are included in the rqJWT to ensure integrity and authenticity of the original request. Each downstream recipient receives the orJWT and verifies its signature before processing the request based on the information within the rqJWT.</p>
<p>To maintain security and transparency throughout the propagation process, it's essential to follow these steps:</p>
<ol>
  <li>
    <strong>Inclusion of orJWT</strong>: The orJWT, generated by the initial requester (1st Party), is included in each request to each downstream recipient.</li>
  <li>
    <strong>Verification and Processing</strong>:&nbsp;</li>
  <ol>
    <li>Upon receiving the rqJWT, each recipient verifies the signature of the embedded orJWT using the public key published on the domain of the 1st party from which the initial request was made. This ensures that the request originated from the claimed source at the time indicated and has not been tampered with during transmission. &nbsp;</li>
    <li>Additionally, the recipient verifies the signature of the rqJWT, which may have been generated by the 1st party or an intermediary. Once the signatures are verified, the recipient processes the request based on the information contained within the rqJWT.</li>
  </ol>
  <li>
    <strong>Generation of New rqJWT</strong>:&nbsp;</li>
  <ol>
    <li>Should a downstream recipient need to further propagate a request, they generate a new rqJWT which includes the orJWT they received as well as any additional information specific to their relationship with the downstream recipient.&nbsp;</li>
  </ol>
</ol>
<p>This approach allows each downstream recipient to verify the authenticity of the original request and assures the integrity of the data deletion process, while adding any necessary additional information to the request before passing it on.</p>
<h2 id="request-data">Deletion Request Data</h2>
<p>Each request needs to be an atomic unit, which can be individually processed by the recipient. The deletion request JWTs will include the following values:</p>
<h5>orJWT: Originating Requester JWT&nbsp;</h5>
<div>
  <table>
    <tbody>
      <tr>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Field Name</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Type</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Description</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Required</strong>
          </span>
        </td>
      </tr>
      <tr>
        <td>version</td>
        <td>string</td>
        <td>Version of the data format.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>jti</td>
        <td>string</td>
        <td>The
          <a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7">
            <span style="color:rgb(17, 85, 204);">"jti"</span>
          </a>
          (JWT ID) claim provides a unique identifier for the JWT, serving as a global request identifier for tracking and managing the request throughout its lifecycle.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>iss</td>
        <td>string</td>
        <td>The "<a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1">
            <span style="color:rgb(17, 85, 204);">iss</span>
          </a>" (issuer) claim identifies the principal that issued the JWT. It represents the eTLD+1 of the 1st party and can be used to locate their dsrdelete.json file.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>iat</td>
        <td>NumericDate</td>
        <td>The "iat" (issued at) claim identifies the time at which the JWT was issued, representing the timestamp of the deletion request.</td>
        <td>required</td>
      </tr>
    </tbody>
  </table>
</div>
<h5>rqJWT: Requester “request” JWT</h5>
<div>
  <table>
    <tbody>
      <tr>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Field Name</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Type</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Description</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Required</strong>
          </span>
        </td>
      </tr>
      <tr>
        <td>version</td>
        <td>string</td>
        <td>Version of the data format.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>orJWT</td>
        <td>string</td>
        <td>The 1st party "originating requester" JWT or orJWT.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>jti</td>
        <td>string</td>
        <td>The
          <a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7">
            <span style="color:rgb(17, 85, 204);">"jti"</span>
          </a>
          (JWT ID) claim provides a unique identifier for the JWT, identifying the specific deletion request transaction.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>iss</td>
        <td>string</td>
        <td>The "<a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1">
            <span style="color:rgb(17, 85, 204);">iss</span>
          </a>" (issuer) claim identifies the principal that issued the JWT. It represents the eTLD+1 of the requester, which can be used to locate their dsrdelete.json file.</td>
        <td>required</td>
      </tr>
      <tr>
      <td>identifierValue</td>
      <td>string</td>
      <td>The value of the identifier to which the deletion request applies.</td>
      <td>required</td>
      </tr>
      <tr>
      <td>identifierType</td>
      <td>string</td>
      <td>The type of identifier provided in the identifierValue field. The value in this field must match a type specified in the intended recipients' dsrdelete.json file.</td>
      <td>required</td>
      </tr>
      <tr>
      <td>identifierFormat</td>
      <td>string</td>
      <td>The format of the identifier provided in the identifierValue field. The value in this field must match a format specified in the intended recipients' dsrdelete.json file.</td>
      <td>required</td>
      </tr>
      <tr>
        <td>iat</td>
        <td>NumericDate</td>
        <td>The "iat" (issued at) claim identifies the time at which the JWT was issued, representing the timestamp of the request transaction.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>optionalParameters</td>
        <td>string</td>
        <td>A JSON object that contains optional parameters. The structure and content of this object are defined by the parties involved in transactions that utilize it.</td>
        <td>optional</td>
      </tr>
    </tbody>
  </table>
</div>
<h5>acJWT: Recipient “acknowledgement” JWT</h5>
<div>
  <table>
    <tbody>
      <tr>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Field Name</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Type</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Description</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Required</strong>
          </span>
        </td>
      </tr>
      <tr>
        <td>version</td>
        <td>string</td>
        <td>Version of the data format.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>rqJWT</td>
        <td>string</td>
        <td>The requester “request” JWT or rqJWT.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>jti</td>
        <td>string</td>
        <td>The
          <a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.7">
            <span style="color:rgb(17, 85, 204);">"jti"</span>
          </a>
          (JWT ID) claim provides a unique identifier for the JWT, identifying the specific acknowledgment transaction.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>iss</td>
        <td>string</td>
        <td>The "<a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7519#section-4.1.1">
            <span style="color:rgb(17, 85, 204);">iss</span>
          </a>" (issuer) claim identifies the principal that issued the JWT. It represents the eTLD+1 of the recipient, which can be used to locate their dsrdelete.json file.
        </td>
        <td>required</td>
      </tr>
      <tr>
        <td>iat</td>
        <td>NumericDate</td>
        <td>The "iat" (issued at) claim identifies the time at which the JWT was issued, representing the timestamp of the acknowledgement transaction.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>raResultCode</td>
        <td>Numeric</td>
        <td>A code indicating the request was successfully validated and accepted or identifying an error.</td>
        <td>required</td>
      </tr>
      <tr>
        <td>raResultString</td>
        <td>string</td>
        <td>Additional information related to the result, such as error details.</td>
        <td>optional</td>
      </tr>
    </tbody>
  </table>
</div>
<h2>Example Deletion Packet</h2>
<p>A data deletion request is accomplished by sending a POST request to the data deletion request API, which returns a response in JSON format. This protocol is implemented via a standard API and request format with a defined JSON payload as a JWT.&nbsp;</p>
<h5>Example Request with required parameters:</h5>
<p>
  <em>The following example includes a decoded orJWT to show the header &amp; payload JSON.</em>
</p>

```
{
    "typ": "JWT",
    "alg": "RS256",
    "kid": "abc123"
}
.
{
    "version": "2.0",
    "jti": "unique_jwt_identifier",
    "iss": "publisher1.com",
    "iat": 1693459424
}
```

<h5>Example Request including optional parameters:</h5>
<p>
  <em>The following example includes a decoded rqJWT to show the header &amp; payload JSON.</em>
</p>


```
{
    "typ": "JWT",
    "alg": "RS256",
    "kid": "abc123"
}
.
{
    "version": "2.0",
    "orJWT": "base64url_header.base64url_payload.base64url_signature",
    "jti": "unique_jwt_identifier",
    "iss": "vendor1.com",
    "identifierValue": "28f6dc889e...fe167",
    "identifierType": "email",
    "identifierFormat": "sha256"
    },
    "iat": 1693459424,
    "optionalParameters": {optional_parameters_information}
}
```

<h5>Example Acknowledgement</h5>
<p>
  <em>The following example includes a decoded acJWT to show the header &amp; payload JSON.</em>
</p>

```
{
    "typ": "JWT",
    "alg": "RS256",
    "kid": "abc123"
}
.
{
    "version": "2.0",
    "rqJWT": "original_jwt_information",
    "jti": "6G7H8J",
    "iss": "vendor2.com",
    "iat": 1693459424,
    "raResultCode": 14201,
    "raResultString": "rq_unsupported_identifier_type"
}
```

<h4>Result Codes</h4>
<p>When requests are processed successfully, recipients are expected to respond with an HTTP 202 status code indicating the request was accepted. In cases where an error is encountered, recipients should respond with an HTTP 400 status code to indicate the failure. Additionally, recipients should include the defined Result Code of the error encountered in the acJWT response payload raResultCode claim and optionally a string with additional details about the error in the acJWT raResultString claim.</p>
<p>For guidelines on error handling, please refer to the following table:</p>
<div>
  <table>
  <thead>
    <tr>
      <th>Result Code</th>
      <th>Result String</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>10000</td>
      <td>success</td>
      <td>Successful: Recipient has acknowledged successful receipt of the deletion request.</td>
    </tr>
    <tr>
      <td>10010</td>
      <td>unknown_error</td>
      <td>Unknown error: An unexpected error occurred that does not correspond to any defined error condition.</td>
    </tr>
    <tr>
      <td>11101</td>
      <td>or_domain_connection_failed</td>
      <td>Could not connect to the domain listed in the orJWT iss claim: <code>{orJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11102</td>
      <td>or_dsrdelete_file_not_found</td>
      <td>Could not find the dsrdelete.json file for the orJWT issuer: <code>{orJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11103</td>
      <td>or_jwksuri_not_found</td>
      <td>Could not find jwksUri entry in the dsrdelete.json for the orJWT issuer: <code>{orJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11104</td>
      <td>or_jwks_keys_file_not_found</td>
      <td>Could not find keys file: <code>{or dsrdelete.json jwksUri}</code> identified in the jwksUri entry in the dsrdelete.json in the orJWT issuer: <code>{orJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11105</td>
      <td>or_jwks_kid_not_found</td>
      <td>Could not find the key with kid: <code>{or kid}</code> in keys file identified: <code>{or dsrdelete.json jwksUri}</code> identified in the jwksUri entry in the dsrdelete.json for the orJWT issuer: <code>{orJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11201</td>
      <td>rq_domain_connection_failed</td>
      <td>Could not connect to the domain listed in rqJWT iss claim: <code>{rqJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11202</td>
      <td>rq_dsrdelete_file_not_found</td>
      <td>Could not find the dsrdelete.json file for the rqJWT issuer: <code>{rqJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11203</td>
      <td>rq_jwksuri_not_found</td>
      <td>Could not find jwksUri entry in the dsrdelete.json for the rqJWT issuer: <code>{rqJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11204</td>
      <td>rq_jwks_keys_file_not_found</td>
      <td>Could not find keys file: <code>{rq dsrdelete.json jwksUri}</code> identified in the jwksUri entry in the dsrdelete.json in the rqJWT issuer: <code>{rqJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>11205</td>
      <td>rq_jwks_kid_not_found</td>
      <td>Could not find the key with kid: <code>{rq kid}</code> in keys file identified: <code>{rq dsrdelete.json jwksUri}</code> identified in the jwksUri entry in the dsrdelete.json for the rqJWT issuer: <code>{rqJWT iss}</code>.</td>
    </tr>
    <tr>
      <td>12101</td>
      <td>or_alg_claim_mismatch</td>
      <td>Request kid: <code>{orJWT kid}</code> alg claim: <code>{orJWT alg}</code> does not match the alg claim: <code>{key file alg}</code> in the keys file: <code>{or dsrdelete.json jwksUri}</code> for the kid.</td>
    </tr>
    <tr>
      <td>12201</td>
      <td>rq_alg_claim_mismatch</td>
      <td>Request kid: <code>{rqJWT kid}</code> alg claim: <code>{rqJWT alg}</code> does not match the alg claim: <code>{key file alg}</code> in the keys file: <code>{rq dsrdelete.json jwksUri}</code> for the kid.</td>
    </tr>
    <tr>
      <td>13101</td>
      <td>or_invalid_token</td>
      <td>Invalid token: The orJWT could not be successfully decoded. JSON token identifier: <code>{orJWT jti}</code>.</td>
    </tr>
    <tr>
      <td>13102</td>
      <td>or_invalid_token_signature</td>
      <td>Invalid token signature: The signature provided in the orJWT is invalid. JSON token identifier: <code>{orJWT jti}</code>.</td>
    </tr>
    <tr>
      <td>13103</td>
      <td>or_invalid_token_timestamp</td>
      <td>Invalid token timestamp: The timestamp provided in the orJWT is invalid: <code>{orJWT timestamp}</code>. The timestamp is a value which is not a positive integer.</td>
    </tr>
    <tr>
      <td>13104</td>
      <td>or_invalid_token_timestamp</td>
      <td>Invalid token timestamp: The timestamp provided in the orJWT is invalid: <code>{orJWT timestamp}</code>. The timestamp is too far in the future: issued more than 5 minutes ahead of the token receipt.</td>
    </tr>
    <tr>
      <td>13201</td>
      <td>rq_invalid_token</td>
      <td>Invalid token: The rqJWT could not be successfully decoded. JSON token identifier: <code>{rqJWT jti}</code>.</td>
    </tr>
    <tr>
      <td>13202</td>
      <td>rq_invalid_token_signature</td>
      <td>Invalid token signature: The signature provided in the rqJWT is invalid. JSON token identifier: <code>{rqJWT jti}</code>.</td>
    </tr>
    <tr>
      <td>13203</td>
      <td>rq_invalid_token_timestamp</td>
      <td>Invalid token timestamp: The timestamp provided in the rqJWT is invalid: <code>{rqJWT timestamp}</code>. The timestamp is a value which is not a positive integer.</td>
    </tr>
    <tr>
      <td>13204</td>
      <td>rq_invalid_token_timestamp_future</td>
      <td>Invalid token timestamp: The timestamp provided in the rqJWT is invalid: <code>{rqJWT timestamp}</code>. The timestamp is too far in the future: issued more than 5 minutes ahead of the token receipt.</td>
    </tr>
    <tr>
      <td>13205</td>
      <td>rq_invalid_timestamp_sequence</td>
      <td>Invalid timestamp sequence: The timestamp of rqJWT: <code>{rqJWT timestamp}</code> precedes the timestamp of orJWT: <code>{orJWT timestamp}</code>.</td>
    </tr>
    <tr>
      <td>14201</td>
      <td>rq_unsupported_identifier_type</td>
      <td>Unsupported identifier type: The identifier type in the rqJWT: <code>{rqJWT identifierType}</code> isn't supported.</td>
    </tr>
    <tr>
      <td>14202</td>
      <td>rq_unsupported_identifier_format</td>
      <td>Unsupported identifier format: The identifier format in the rqJWT <code>{rqJWT identifierFormat}</code> isn't supported.</td>
    </tr>
    <tr>
      <td>14203</td>
      <td>rq_incorrect_identifier_format</td>
      <td>Incorrect identifier format: The identifier format in the rqJWT: <code>{rqJT identifierFormat}</code> does not match the format of the received rqJWT's identifierValue.</td>
    </tr>
    <tr>
      <td>14204</td>
      <td>rq_invalid_identifier_value</td>
      <td>Invalid identifier value: The identifier value in the rqJWT is invalid (e.g. incorrect length).</td>
    </tr>
  </tbody>
</table>
</div>
<h2>Identifiers</h2>
<p>There are generally two classes of identifiers used in ad-interactions distinguished by access limitations: those which are directly accessible by 1st-parties and those which are only accessible to 3rd parties. The first class includes any identifiers available in the 1st-party context, including on web pages and 1st-party local storage. The second class includes identifiers maintained in protected 3rd-party storage, such as 3rd-party cookies. Probabilistic identifiers, which are based on constellations of data values, presumably may fall into the first, second or a combination of both classes, depending on how they are constructed.</p>
<h5>Identifier Communication</h5>
<p>The initial version of this proposal only provides support for identifiers a 1st-party has access to prior to initiating communication of the data deletion request. It is the responsibility of the 1st Party to determine what identifiers they must provide to which partners in deletion requests and to provide them in a form usable by those partners for satisfying the request.&nbsp;</p>
<p>The details of what identifiers a participant can accept and what format they can accept them in are provided by each participant in the dsrdelete.json file on their domain. Data deletion requests should contain the same identifiers sent to partners in data transactions. The 1st Party will gather this information from a partners dsrdelete.json files so that each request only includes a supported identifier.</p>
<h2>Request/Response Signatures</h2>
<p>To ensure deletion requests are legitimate and unmodified, participants will cryptographically sign requests and responses with private keys. The corresponding public keys will be published in their JWKS (JSON Web Key Set) files so that other participants can use them to verify signatures.</p>
<h5>Cryptographic Keys</h5>
<p>To accomplish this, this specification will use JSON Web Key Sets (JWKS). The JWKS standard (<a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7517">
    <span style="color:rgb(17, 85, 204);">RFC 7517</span>
  </a>) establishes a way to store and manage cryptographic keys as a set of JSON objects. JWKS supports asymmetric signature with a public key and private key pair, supporting the discoverability design of the <code>dsrdelete.json</code> file and the signature requirements of the data deletion framework.</p>
  <p>Note that while <a href="https://datatracker.ietf.org/doc/html/rfc7517">RFC 7517</a> lists the <code>"alg"</code> and the <code>"kid"</code> parameters as optional, they are required in the context of this framework to ensure all participants can easily locate verification keys.</p>
<h5>Key Rotation and Caching</h5>  
<p>Participants must support key rotation to maintain secure and reliable verification. Implementations should maintain a local cache of each participant's JWKS. Participants must refresh their cached JWKS according to the <code>pollFrequency</code>. When verifying a signed request, if a referenced key ID (<code>kid</code>) cannot be found in the cached key set, the participant must fetch a new copy of the requester's JWKS from the <code>jwksUri</code> location and update its cache accordingly.</p>
<h5>JWKS Resources</h5>
<p>For more information on JWKS format, please reference the following resources:</p>
<ul>
  <li>IETF JWKS Standard:
    <a target="_blank" href="https://datatracker.ietf.org/doc/html/rfc7517#appendix-A">
      <span style="color:rgb(17, 85, 204);">RFC 7517 - JSON Web Key Set (JWKS)</span>
    </a>
  </li>
</ul>
<h2>Discovery</h2>
<p>This specification defines a standard method by which participants communicate what information they require to execute data deletion requests and resources needed by other participants when interacting with them, such as cryptographic keys. To achieve this, there will be two options of discovery for participants. Both options are available to support multiple use-cases for discovery. To achieve this, participants will be required to create a dsrdelete.json file, which will be considered the authoritative version of a participant's information.</p>
<h4>dsrdelete.json resource</h4>
<p>For easy access and discovery, participants must provide a JSON file named dsrdelete.json in the root of their domain. Parties supporting this standard would be expected to periodically read the files published by their partners and assure they support what is required by those partners in order to honor requests. Having participants publish this information directly on their domains allows them to fully and directly manage it, allows partners to work directly with them to resolve issues, and eliminates the potential for a single point of failure to disable communications.&nbsp;</p>
<p>1st Parties should read the data provided in the well-known resource from the vendor. This resource should include the following:</p>
<h4>Example Resource</h4>
<div>
  <table>
    <tbody>
      <tr>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Field Name</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Type</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Description</strong>
          </span>
        </td>
        <td>
          <span style="color:rgb(255, 255, 255);">
            <strong>Required/Optional</strong>
          </span>
        </td>
      </tr>
      <tr>
        <td>endpoint</td>
        <td>string</td>
        <td>API endpoint for deletion requests.</td>
        <td>Required</td>
      </tr>
       <tr>
        <td>pollFrequency</td>
        <td>number </td>
        <td>Polling frequency, in seconds</td>
        <td>Required</td>
      </tr>
      <tr>
        <td>identifiers</td>
        <td>Array of object</td>
        <td>List of supported identifiers.</td>
        <td>Required</td>
      </tr>
      <tr>
        <td>jwksUri</td>
        <td>string</td>
        <td>Location of the JSON Web Key Set (JWKS) endpoint which contains the JWKS used to sign all issued JSON Web Tokens (JWTs).</td>
        <td>Required</td>
      </tr>
      <tr>
        <td>publishedJwksUri</td>
        <td>string</td>
        <td>Location of the endpoint which contains prior JWK sets that include previously active signing keys, to facilitate signature verification of tokens issued before key rotation.</td>
        <td>Optional</td>
      </tr>
      <tr>
        <td>vendorScriptRequirement</td>
        <td>boolean</td>
        <td>True or false for requirement to use vendor script</td>
        <td>Required</td>
      </tr>
      <tr>
        <td>vendorScript</td>
        <td>string</td>
        <td>Optional field for vendors to publish a HTML code (e.g. a &lt;script&gt;) if this is needed to gather an identifier for a deletion request.</td>
        <td>Optional</td>
      </tr>
      <tr>
        <td>dsrdeleteJsonUri</td>
        <td>string</td>
        <td>Optional field for partners with multiple domains who only wish for one to host the complete file.</td>
        <td>Optional</td>
      </tr>
      <tr>
        <td>optionalParameters</td>
        <td>Array of objects</td>
        <td>Optional field for implementers to add custom, optional fields</td>
        <td>Optional</td>
      </tr>
    </tbody>
  </table>
</div>
<h4>Example Resource for example.com/.well-known/iabtl/dsrdelete.json:</h4>

```
https://www.publisher1.com/dsrdelete.json

{
    "endpoint": "https://www.publisher1.com/api/delete/",
    "pollFrequency": "259200", // 30 days
    "identifiers": [
        { "id": 1, "type": "email", "format": "sha256" },
        { "id": 2, "type": "idfa", "format": "hash" }
    ],
    "jwksUri": "https://example.com/.well-known/iabtl/jwks.json",
    "vendorScriptRequirement": true,
    "vendorScript": "<script>.....</script>"
}
```

<h4>Example Resource for example.com/.well-known/iabtl/jwks.json</h4>

```
{
    "pollFrequency": "604800", // 7 days
    "keys": [
            {
                "kty": "EC",
                "alg": "ES256",
                "crv": "P-256",
                "x": "8PdtOAYA6bYKVDWZ16kzAlQkS9JEZakZ3rmXbzIl0nk",
                "y": "D2qikPEHzo6Q9qLghINZ5nO8u8zjcRNbZnbW7M6pJr0",
                "kid": "unique_identifier_for_1st_key"

            },
		        {
                "kty": "EC",
                "alg": "ES256",
                "crv": "P-256",
                "x": "RyIDwX4y7tkctGt65W0WaslpQ9j9AnHcUiKhsKrcMWw",
                "y": "kRbxhHFRLjd75Xa3Ds_U196o4lBHWkhyq-YKoCQPAfU",
                "kid": "unique_identifier_for_2nd_key"
            },
		        {
                "kty": "EC",
                "alg": "ES256",
                "crv": "P-256",
                "x": "SLvexvatx17wweJkO4GwhOKaa_Z7DViYarzpLIAl3dE",
                "y": "SgP1HvjKhhqQ8nqpAKTD_WBq6EcBt40jRsbbXRfjU7E",
                "kid": "unique_identifier_for_3rd_key"
            }
            ]
}
```

<h4>Implementing Script-based deletion requests</h4>
<p>Participants that only support script based deletion requests, can do so by providing a property vendorScript along with a property vendorScriptRequirement in their dsrdelete.json file. A first party that wants to start the deletion process, will execute the script.</p>
<p>
  <strong>Note</strong>: Please note that supporting only script based deletion requests will limit the ability to receive deletion requests where requests are chained via endpoints.</p>
<h4>Directory</h4>
<p>An optional discovery method will be available via access to a directory provided by the Tech Lab. The directory is available in the <a href="https://tools.iabtechlab.com/transparencycenter/explorer/business/datadeletionregistry">Tech Lab tools portal</a> and provide participants with the location of each dsrdelete.json resource. To create the directory entries, Tech Lab will crawl dsrdelete.json resources and automatically create entries–similar to ads.txt. The results would appear in the Tools Portal and in an API. Participants might use the directory to look up primary domains for companies that operate across multiple domains, validate a vendor has adopted the standard with the established request format, or to simplify a bulk lookup of endpoints from participants. All endpoints can be looked up, but not all endpoints need to be used.</p>
