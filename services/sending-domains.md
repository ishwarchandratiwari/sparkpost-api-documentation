title: Sending Domains
description: Manage sending domains, which are used to indicate who an email is from.

# Group Sending Domains
<a name="sending-domains-api"></a>

A sending domain is a domain that is used to indicate who an email is from via the "From:" header. Using a custom sending domain enables you to control what recipients see as the From value in their email clients. DNS records can be configured for a sending domain, which allows recipient mail servers to authenticate your messages. The Sending Domains API provides the means to create, list, retrieve, update, and verify a custom sending domain.

<div class="alert alert-danger"><strong>For maximum deliverability</strong>, we recommend <a href="sending-domains.html#sending-domains-verify-post">configuring</a> DKIM for your sending domains <em>and</em> <a href="https://www.sparkpost.com/docs/tech-resources/custom-bounce-domain/">configuring a bounce domain</a> on corresponding subdomains. This is an easy way to help mailbox providers authenticate and differentiate your email from other senders using SparkPost.</div>

<div class="alert alert-info"><strong>Note</strong>: When adding a sending domain to your account, the domain must be verified within two weeks or it will be removed from your account.</div>

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://app.getpostman.com/run-collection/5d9ae743a661a15d64bb)

## Sending Domain Attributes

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|domain    | string | Name of the sending domain | yes |The domain name will be used as the "From:" header address in the email.|
|tracking_domain | string | Associated tracking domain | no | example: "click.example1.com"<br/><span class="label label-info"><strong>Note</strong></span> tracking domain and sending domain must belong to the same subaccount to be linked together.|
|status | JSON object | JSON object containing status details, including whether this domain's ownership has been verified  | no | Read only. For a full description, see the [Status Attributes](#header-status-attributes).|
|dkim | JSON object | JSON object in which DKIM key configuration is defined | no | For a full description, see the [DKIM Attributes](#header-dkim-attributes).|
|generate_dkim | boolean | Whether to generate a DKIM keypair on creation | no | Defaults to `true` |
|dkim_key_length | number | Size, in bits, of the DKIM private key to be generated  | no | This option only applies if generate_dkim is 'true'. Private key size defaults to 1024.<br/><span class="label label-info"><strong>Note</strong></span> public keys for private keys longer than 1024 bits will be longer that 255 characters.  Because of this, the public key `TXT` record in DNS will need to contain multiple strings, see [RFC 7208, section 3.3](https://tools.ietf.org/html/rfc7208#section-3.3) for an example of how the SPF spec addresses this.|
|shared_with_subaccounts | boolean | Whether this domain can be used by subaccounts | no | Defaults to `false`.  Only available to domains belonging to a master account.|
|is_default_bounce_domain | boolean | Whether this domain should be used as the bounce domain when no other valid bounce domain has been specified in the transmission or SMTP injection | no | Defaults to `false`.  Only available to domains with cname_status of "valid" or mx_status of "valid".  The master account as well as each subaccount may set a unique default bounce domain.<br><br>Not available in <span class="label label-warning"><strong>Enterprise</strong></span>|
|creation_time	| string | Datetime the domain was created | no | Read only. Format: YYYY-MM-DDTHH:MM:SS+-HH:MM|
|delegated | boolean | Whether this domain was delegated to SparkPost by the customer | no | Defaults to `false`. Read only. Will not be present if false. <br><br>Only available in <span class="label label-warning"><strong>Enterprise</strong></span>  |

### DKIM Attributes

DKIM uses a pair of public and private keys to authenticate your emails. PKCS #1 and PKCS #8 formats are supported. We do not support password-protected keys.

<div class="alert alert-info"><strong>Note</strong>: The public/private key pair must match a single format as the API will reject mismatching pairs.</div>

The DKIM key configuration is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|signing_domain| string | Signing Domain Identifier (SDID) | no |This will be used in the `d=` field of the DKIM Signature. If `signing_domain` is not specified, or is set to the empty string (""), then the Sending Domain will be used as the signing domain.<br/>By default, SparkPost uses the Sending Domain as the signing domain. <br><br>Only writable in <span class="label label-warning"><strong>Enterprise</strong></span> |
|private | string | DKIM private key | yes | The private key will be used to create the DKIM Signature.|
|public | string |DKIM public key  | yes | The public key will be retrieved from DNS of the sending domain.|
|selector | string |DomainKey selector | yes | The DomainKey selector will be used to indicate the DKIM public key location.|
|headers | string| Header fields to be included in the DKIM signature |no | **This field is currently ignored.** |

### Status Attributes

Detailed status for this sending domain is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Default   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|ownership_verified | boolean | Whether domain ownership has been verified |false |Read only. This field will return `true` if any of dkim_status, cname_status, mx_status, spf_status, abuse_at_status, postmaster_at_status, or verification_mailbox_status are `true` or ownership has been verified previously.|
|dkim_status | string | Verification status of DKIM configuration |unverified|Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.|
|cname_status | string | Verification status of CNAME configuration |unverified |Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.|
|mx_status | string | Verification status of MX configuration |unverified |Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.<br><br>Only available in <span class="label label-warning"><strong>Enterprise</strong></span> |
|spf_status | string | Verification status of SPF configuration |unverified |Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.  <span class="label label-danger"><strong>Deprecated</strong></span>|
|abuse_at_status | string | Verification status of abuse@ mailbox |unverified |Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.|
|postmaster_at_status | string | Verification status of postmaster@ mailbox |unverified |Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.|
|verification_mailbox_status | string | Verification status of nominated anyone@ mailbox |unverified |Read only. Valid values are `unverified`, `pending`, `invalid` or `valid`.|
|verification_mailbox | string | Nominated anyone@ verification mailbox email address local part | |Read only. This field will only be returned if it was set on a prior POST to verify a sending domain using verification_mailbox.|
|compliance_status | string | Compliance status | | Valid values are `pending`, `valid`, or `blocked`.|

### Verify Attributes

These are the valid request options for verifying a Sending Domain:


| Field         | Type     | Description                           | Required  | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|dkim_verify | boolean | Request verification of DKIM record | no | |
|cname_verify | boolean | Request verification of CNAME record | no | CNAME verification is a pre-requisite for the domain to be used as a bounce domain.  See the [verify endpoint](#sending-domains-verify-post). |
|verification_mailbox_verify | boolean | Request an email with a verification link to be sent to a nominated mailbox on the sending domain. | no | The nominated mailbox is specified in the verification_mailbox field.  The mailbox can be any valid mailbox for the domain other than "postmaster" or "abuse".  Not available in <span class="label label-warning"><strong>Enterprise</strong></span> |
|verification_mailbox | string | The nominated mailbox email address local part to be used when requesting email with a verification link be sent. | no | Required if "verification_mailbox_verify" = true. Not available in <span class="label label-warning"><strong>Enterprise</strong></span> |
|postmaster_at_verify | boolean | Request an email with a verification link to be sent to the sending domain's postmaster@ mailbox. | no | |
|abuse_at_verify | boolean | Request an email with a verification link to be sent to the sending domain's abuse@ mailbox. | no | |
|verification_mailbox_token | string | A token retrieved from the verification link contained in the verification email. | no | <br><br>Not available in <span class="label label-warning"><strong>Enterprise</strong></span>|
|postmaster_at_token | string | A token retrieved from the verification link contained in the postmaster@ verification email. | no | |
|abuse_at_token | string | A token retrieved from the verification link contained in the abuse@ verification email. | no | |

### DNS Attributes

| Field         | Type     | Description                           |
|------------------------|:-:       |---------------------------------------|
|dkim_record | string | DNS DKIM record for the registered Sending Domain |
|cname_record | string | DNS CNAME record for the registered Sending Domain |
|dkim_error | string | Error message describing reason for DKIM verification failure |
|cname_error | string | Error message describing reason for CNAME verification failure |

## Create [/sending-domains]

### Create a Sending Domain [POST]

Create a sending domain by providing a **sending domain object** as the POST request body.

We allow any given domain (including its subdomains) to only be used by a single customer account.  Please see our [support article](https://support.sparkpost.com/customer/en/portal/articles/1933318-creating-sending-domains) for additional reasons a domain might not be approved for sending.

To use a DKIM Signing Domain Identifier different to the Sending Domain, set the <tt>dkim.signing_domain</tt> field.

<div class="alert alert-info"><strong><a href="https://www.sparkpost.com/enterprise-email/">SparkPost Enterprise</a></strong> accounts: In some configurations, Sending Domains will be verified automatically when they are created, and can be used to send messages immediately. In that case, there is no need separately verify Sending Domains. To find out if this applies to your SparkPost Enterprise service, please ask your TAM or contact Support through the app.
</div>

+ Request Create New Sending Domain with Auto-Generated DKIM Keypair (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

           {
               "domain": "example1.com",
               "tracking_domain": "click.example1.com",
               "generate_dkim": true,
               "shared_with_subaccounts": false

           }

+ Response 200 (application/json)

        {
          "results": {
            "message": "Successfully Created domain.",
            "domain": "example1.com",
            "dkim": {
              "public": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
              "selector": "scph0316",
              "signing_domain": "example1.com",
              "headers": "from:to:subject:date"
            }
          }
        }

+ Request Create Sending Domain without DKIM Keypair

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

           {
               "domain": "example1.com",
               "generate_dkim": false
           }

+ Response 200 (application/json)

        {
          "results": {
            "message": "Successfully Created domain.",
            "domain": "example1.com"
          }
        }

+ Request Provide Pre-Generated DKIM Keypair (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

          {
              "domain": "example1.com",
              "tracking_domain": "click.example1.com",
              "dkim": {  "private": "MIICXgIBAAKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQABAoGBAITb3BCRPBi5lGhHdn+1RgC7cjUQEbSb4eFHm+ULRwQ0UIPWHwiVWtptZ09usHq989fKp1g/PfcNzm8c78uTS6gCxfECweFCRK6EdO6cCCr1cfWvmBdSjzYhODUdQeyWZi2ozqd0FhGWoV4VHseh4iLj36DzleTLtOZj3FhAo1WJAkEA68T+KkGeDyWwvttYtuSiQCCTrXYAWTQnkIUxduCp7Ap6tVeIDn3TaXTj74UbEgaNgLhjG4bX//fdeDW6PaK9YwJBAM6xJmwHLPMgwNVjiz3u/6fhY3kaZTWcxtMkXCjh1QE82KzDwqyrCg7EFjTtFysSHCAZxXZMcivGl4TZLHnydJUCQQCx16+M+mAatuiCnvxlQUMuMiSTNK6Amzm45u9v53nlZeY3weYMYFdHdfe1pebMiwrT7MI9clKebz6svYJVmdtXAkApDAc8VuR3WB7TgdRKNWdyGJGfoD1PO1ZE4iinOcoKV+IT1UCY99Kkgg6C7j62n/8T5OpRBvd5eBPpHxP1F9BNAkEA5Nf2VO9lcTetksHdIeKK+F7sio6UZn0Rv7iUo3ALrN1D1cGfWIh2dj3ko1iSreyNVSwGW0ePP27qDmU+u6/Y1g==",
                  "public": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                  "selector": "scph0316",
                  "headers": "from:to:subject:date"
              }
          }

+ Response 200 (application/json)

        {
          "results": {
            "message": "Successfully Created domain.",
            "domain": "example1.com",
            "dkim": {
              "public": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
              "selector": "scph0316",
              "signing_domain": "example1.com",
              "headers": "from:to:subject:date"
            }
          }
        }

+ Response 400 (application/json)

           {
             "errors": [
               {
                 "message": "invalid params",
                 "description": "Tracking domain 'click.example1.com' is not a registered tracking domain",
                 "code": "1200"
               }
             ]
           }

+ Response 422 (application/json)

           {
             "errors": [
               {
                 "message": "invalid data format/type",
                 "description": "Error validating domain name syntax for domain: '(domain)'",
                 "code": "1300"
               }
             ]
           }

## List [/sending-domains{?ownership_verified,dkim_status,cname_status,mx_status,abuse_at_status,postmaster_at_status,compliance_status,is_default_bounce_domain}]

### List all Sending Domains [GET]

List an overview of all sending domains in the system.  By default, all domains are returned.  Use the query parameters to filter on the various status options.

+ Parameters
    + ownership_verified (optional, boolean, `true`) ... Ownership verified flag.  Valid values are `true` or `false`.  If not provided, returns a list of all domains regardless of ownership verification.
    + dkim_status (optional, string, `valid`) ... DKIM status filter.  Valid values are `valid`, `invalid`, `unverified`, or `pending`.  If not provided, returns a list of all domains regardless of DKIM status.
    + cname_status (optional, string, `valid`) ... CNAME status filter.  Valid values are `valid`, `invalid`, `unverified`, or `pending`.  If not provided, returns a list of all domains regardless of CNAME status.
    + mx_status (optional, string, `unverified`) ... MX status filter.  Valid values are `valid`, `invalid`, `unverified`, or `pending`.  If not provided, returns a list of all domains regardless of MX status.
    + abuse_at_status (optional, string, `unverified`) ... abuse@ status filter.  Valid values are `valid`, `invalid`, `unverified`, or `pending`.  If not provided, returns a list of all domains regardless of abuse@ status.
    + postmaster_at_status (optional, string, `unverified`) ... postmaster@ status filter.  Valid values are `valid`, `invalid`, `unverified`, or `pending`.  If not provided, returns a list of all domains regardless of postmaster@ status.
    + compliance_status (optional, string, `valid`) ... compliance status filter.  Valid values are `valid`, `blocked`, or `pending`.  If not provided, returns a list of all domains regardless of compliance status.
    + is_default_bounce_domain (optional, boolean, `false`) ... Is default bounce domain flag.  Valid values are `true` or `false`.  If not provided, returns a list of all domains regardless of whether it is the default bounce domain.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
            "results": [
                {
                    "domain": "example1.com",
                    "tracking_domain": "click.example1.com",
                    "status": {
                        "ownership_verified": true,
                        "spf_status": "unverified",
                        "abuse_at_status": "unverified",
                        "dkim_status": "valid",
                        "cname_status": "valid",
                        "mx_status": "unverified",
                        "compliance_status": "valid",
                        "postmaster_at_status": "unverified",
                        "verification_mailbox_status": "valid",
                        "verification_mailbox": "susan.calvin"
                    },
                    "shared_with_subaccounts": false,
                    "is_default_bounce_domain" : false
                },
                {
                    "domain": "example2.com",
                    "status": {
                        "ownership_verified": true,
                        "spf_status": "unverified",
                        "abuse_at_status": "unverified",
                        "dkim_status": "valid",
                        "cname_status": "valid",
                        "mx_status": "unverified",
                        "compliance_status": "valid",
                        "postmaster_at_status": "unverified",
                        "verification_mailbox_status": "unverified"
                    },
                    "shared_with_subaccounts": false,
                    "is_default_bounce_domain" : false
                }
            ]
        }

## Retrieve, Update, and Delete [/sending-domains/{domain}]

### Retrieve a Sending Domain [GET]

Retrieve a sending domain by specifying its domain name in the URI path.  The response includes details about its DKIM key configuration.

+ Parameters
  + domain (required, string, `example1.com`) ... Name of the domain

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
            "results": {
                "tracking_domain": "click.example1.com",
                "status": {
                    "ownership_verified": false,
                    "spf_status": "unverified",
                    "abuse_at_status": "unverified",
                    "dkim_status": "unverified",
                    "cname_status": "unverified",
                    "mx_status": "pending",
                    "compliance_status": "pending",
                    "postmaster_at_status": "unverified",
                    "verification_mailbox_status": "unverified"
                },
                "dkim": {
                    "headers": "from:to:subject:date",
                    "public": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                    "selector": "hello_selector"
                },
                "shared_with_subaccounts": false,
                "is_default_bounce_domain" : false
            }
        }

### Update a Sending Domain [PUT]

Update the attributes of an existing sending domain by specifying its domain name in the URI path and use a **sending domain object** as the PUT request body.

If a tracking domain is specified, it will replace any currently specified tracking domain.  If the supplied tracking domain is a blank string, it will clear any currently specified tracking domain. Note that if a tracking domain is not specified, any currently specified tracking domain will remain intact.

If a DKIM object is provided in the update request, it must contain all relevant fields whether they are being changed or not.  The new DKIM object will completely overwrite the existing one.

To remove the DKIM Signing Domain Identifier for a Sending Domain, use an empty string for the value of the <tt>dkim.signing_domain</tt> field.


+ Parameters
    + domain (required, string, `example1.com`) ... Name of the domain

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

           {
               "tracking_domain": "click.example1.com",
               "dkim": {
                   "private": "MIICXgIBAAKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQABAoGBAITb3BCRPBi5lGhHdn+1RgC7cjUQEbSb4eFHm+ULRwQ0UIPWHwiVWtptZ09usHq989fKp1g/PfcNzm8c78uTS6gCxfECweFCRK6EdO6cCCr1cfWvmBdSjzYhODUdQeyWZi2ozqd0FhGWoV4VHseh4iLj36DzleTLtOZj3FhAo1WJAkEA68T+KkGeDyWwvttYtuSiQCCTrXYAWTQnkIUxduCp7Ap6tVeIDn3TaXTj74UbEgaNgLhjG4bX//fdeDW6PaK9YwJBAM6xJmwHLPMgwNVjiz3u/6fhY3kaZTWcxtMkXCjh1QE82KzDwqyrCg7EFjTtFysSHCAZxXZMcivGl4TZLHnydJUCQQCx16+M+mAatuiCnvxlQUMuMiSTNK6Amzm45u9v53nlZeY3weYMYFdHdfe1pebMiwrT7MI9clKebz6svYJVmdtXAkApDAc8VuR3WB7TgdRKNWdyGJGfoD1PO1ZE4iinOcoKV+IT1UCY99Kkgg6C7j62n/8T5OpRBvd5eBPpHxP1F9BNAkEA5Nf2VO9lcTetksHdIeKK+F7sio6UZn0Rv7iUo3ALrN1D1cGfWIh/Y1g==",
                   "public": "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB",
                   "selector": "hello_selector",
                   "headers": "from:to:subject:date"
               },
               "is_default_bounce_domain": true
           }

+ Response 200 (application/json)

        {
            "results": {
                "message": "Successfully Updated Domain.",
                "domain": "example1.com"
            }
        }

+ Response 400 (application/json)

           {
             "errors": [
               {
                 "message": "invalid params",
                 "description": "Tracking domain '(domain)' is not a registered tracking domain",
                 "code": "1200"
               }
             ]
           }

+ Response 422 (application/json)

           {
             "errors": [
               {
                 "message": "invalid data format/type",
                 "description": "Error validating domain name syntax for domain: '(domain)'",
                 "code": "1300"
               }
             ]
           }

### Delete a Sending Domain [DELETE]

Delete an existing sending domain.

<div class="alert alert-danger"><strong>Warning</strong>: Before deleting a sending domain please ensure you are no longer using it. After deleting a sending domain, any new transmissions that use it will result in a rejection. This includes any transmissions that are in progress, scheduled for the future, or use a stored template referencing the sending domain.</div>

+ Parameters
  + domain (required, string, `example1.com`) ... Name of the domain

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 204

+ Response 404 (application/json)

  + Body

            {
              "errors": [
                {
                  "code": "1600",
                  "message": "resource not found",
                  "description": "Domain 'wrong.domain' does not exist"
                }
              ]
            }


## Verify [/sending-domains/{domain}/verify]

### Verify a Sending Domain [POST]

The verify resource operates differently depending on the provided request fields:
  * Including the fields `dkim_verify` or `cname_verify` in the request initiates a check against the associated DNS record type for the specified sending domain.
  * Including the fields `postmaster_at_verify` and/or `abuse_at_verify` in the request results in an email sent to the specified sending domain's postmaster@ and/or abuse@ mailbox where a verification link can be clicked.
  * Including the fields `verification_mailbox_verify` and `verification_mailbox` in the request results in an email sent to the specified mailbox where a verification link can be clicked.
  * For `postmaster_at_verify`, `abuse_at_verify` and `verification_mailbox_verify` ownership verification, if the request is made a 2nd time another email will be sent with a new verification link. If the link in the previously sent message is subsequently clicked it will not verify domain ownership. However, if the link in the most recent email is clicked it will verify domain ownership.
  * Including the fields `verification_mailbox_token` and/or `postmaster_at_token` and/or `abuse_at_token` in the request initiates a check of the provided token(s) against the stored token(s) for the specified sending domain.

**DKIM** public key verification requires the following:
  * A valid DKIM record must be in the DNS for the sending domain being verified.
  * The record must use the sending domain's public key in the `p=` tag.
  * If a k= tag is defined, it must be set to `rsa`.
  * If an h= tag is defined, it must be set to `sha256`.

For example, here is what a DKIM record might look like for domain *mail<span></span>.example.com* with selector *scph1015*:

| Hostname         | Type     | Value                           |
|------------------------|:-:       |---------------------------------------|
|scph1015._domainkey.mail.example.com | TXT | v=DKIM1; k=rsa; h=sha256; p=MIGfMA0GCSqGSIb3DQEBAQUAA5GNADCBiQKBgQCzMTqqPX9jry+nKZjqYhKt5CP4+vBoEpf24POjc5ubWJQnZmY0wdBXawskxC7mBekUlAjOcsbZIhnFt+2asb1XTyLcTjGyqMvVcoUou6olzfMnfB06W9awRahQrrs9E0LZ4hYKSBDTm3MvoJo004+dNpTSnTlGqMyOoBuiD6KX8QIDAQAB |

**CNAME** verification requires the following:
  * <strong>SparkPost</strong> A valid CNAME record in DNS with value `sparkpostmail.com`
  * <strong>SparkPost EU</strong> A valid CNAME record in DNS with value `eu.sparkpostmail.com`
  * <span class="label label-warning"><strong>Enterprise</strong></span> A valid CNAME record in DNS with value `<public_tenant_id>.mail.e.sparkpost.com`

An example CNAME record for a <strong>SparkPost</strong> customer with domain *mail<span></span>.example.com*:

| Hostname         | Type     | Value                           |
|------------------------|:-:       |---------------------------------------|
|mail<span></span>.example.com | CNAME | sparkpostmail<span></span>.com |

An example CNAME record for a <strong>SparkPost Enterprise</strong> customer with public_tenant_id *foo* and with domain *mail<span></span>.example.com*:

| Hostname         | Type     | Value                           |
|------------------------|:-:       |---------------------------------------|
|mail<span></span>.example.com | CNAME | foo.mail.e.sparkpost<span></span>.com |

**MX** verification is available to <strong>Enterprise</strong> customers only. There is no way to initiate MX verification through the /verify endpoint. Please contact your TAM if you want to verify your domain with MX.</div>

#### Using a Sending Domain as a Bounce Domain
A Sending Domain is eligible to be used as a Bounce Domain if one of the following conditions is met:
* A CNAME record in place and verified via ``"cname_verify":true`
* An MX verified domain (<span class="label label-warning"><strong>Enterprise</strong></span> only)

 Eligible domains may be used as a bounce domain by including it as part of the transmission return_path or SMTP MAIL FROM email address. Bounce domains are used to report bounces, which are emails that were rejected from the recipient server. By adding a bounce domain to your account, you can customize the address that is used for the `Return-Path` header, which is the destination for out of band (OOB) bounces.  For additional details on CNAME-verification, please see this [support article](https://www.sparkpost.com/docs/tech-resources/custom-bounce-domain/).

The domain's `status` object is returned on success.

+ Parameters
  + domain (required, string, `example1.com`) ... Name of the domain

+ Request Verify DKIM (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

           {
               "dkim_verify": true
           }


+ Response 200 (application/json)

        {
            "results": {
                "ownership_verified": true,
                "dns": {
                    "dkim_record": "k=rsa; h=sha256; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+W6scd3XWwvC/hPRksfDYFi3ztgyS9OSqnnjtNQeDdTSD1DRx/xFar2wjmzxp2+SnJ5pspaF77VZveN3P/HVmXZVghr3asoV9WBx/uW1nDIUxU35L4juXiTwsMAbgMyh3NqIKTNKyMDy4P8vpEhtH1iv/BrwMdBjHDVCycB8WnwIDAQAB"
                },
                "dkim_status": "valid",
                "cname_status": "unverified",
                "mx_status": "unverified",
                "compliance_status": "pending",
                "spf_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified",
                "verification_mailbox_status": "unverified"
            }
        }

+ Request Verify CNAME (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

           {
               "cname_verify": true
           }


+ Response 200 (application/json)

        {
            "results": {
                "ownership_verified": true,
                "dns": {
                    "cname_record": "sparkpostmail.com"
                },
                "dkim_status": "unverified",
                "cname_status": "valid",
                "mx_status": "unverified",
                "compliance_status": "pending",
                "spf_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified",
                "verification_mailbox_status": "unverified"
            }
        }

+ Request Initiate postmaster@ email (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

           {
               "postmaster_at_verify": true
           }

+ Response 200 (application/json)

        {
            "results": {
                "ownership_verified": false,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "cname_status": "unverified",
                "mx_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified",
                "verification_mailbox_status": "unverified"
            }
        }

+ Request Verify postmaster@ correct token (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

           {
               "postmaster_at_token": "rcayptmrczdnrnqfsxyrzljmtsxvjzxb"
           }

+ Response 200 (application/json)

        {
            "results": {
                "ownership_verified": true,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "cname_status": "unverified",
                "mx_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "valid",
                "verification_mailbox_status": "unverified"
            }
        }

+ Request Initiate anyone@ email (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "verification_mailbox_verify": true,
            "verification_mailbox": "susan.calvin"
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "ownership_verified": false,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified",
                "verification_mailbox_status": "unverified",
                "verification_mailbox": "susan.calvin"
            }
        }

+ Request Verify anyone@ correct token (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "verification_mailbox_token": "bxzjvxstmjlzryxsfqnrndzcrmtpyacr"
        }
        ```

+ Response 200 (application/json; charset=utf-8)

        {
            "results": {
                "ownership_verified": true,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified",
                "verification_mailbox_status": "valid",
                "verification_mailbox": "susan.calvin"
            }
        }

+ Request Initiate anyone@ email without verification_mailbox (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "verification_mailbox_verify": true
        }
        ```

+ Response 422 (application/json; charset=utf-8)

        {
           "errors": [
              {
                 "message": "required field is missing",
                 "description": "verification_mailbox field required to verify mailbox",
                 "code": "1400"
              }
           ]
        }

+ Request Initiate anyone@ email with verification_mailbox set to postmaster (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "verification_mailbox_verify": true,
            "verification_mailbox": "postmaster"
        }
        ```

+ Response 422 (application/json; charset=utf-8)

        {
           "errors": [
              {
                 "message": "invalid data format/type",
                 "description": "verification_mailbox field cannot be 'postmaster' or 'abuse'",
                 "code": "1300"
              }
           ]
        }

+ Request Initiate anyone@ email for sending domain with a DMARC policy (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

        ```
        {
            "verification_mailbox_verify": true,
            "verification_mailbox": "denis.ritchie"
        }
        ```

+ Response 400 (application/json; charset=utf-8)

        {
           "errors": [
              {
                 "message": "Domain not allowed",
                 "description": "Verification by address is not available for Sending Domains with a DMARC policy",
                 "code": "7003"
              }
           ]
        }

+ Request Verify abuse@ incorrect token (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

           {
               "abuse_at_token": "AN_INCORRECT_OR_EXPIRED_TOKEN"
           }

+ Response 200 (application/json)

        {
            "results": {
                "ownership_verified": false,
                "spf_status": "unverified",
                "compliance_status": "valid",
                "dkim_status": "unverified",
                "cname_status": "unverified",
                "mx_status": "unverified",
                "abuse_at_status": "unverified",
                "postmaster_at_status": "unverified",
                "verification_mailbox_status": "unverified"
            }
        }

+ Request Unable to process abuse@ request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
    + Body

           {
               "abuse_at_verify": true
           }

+ Response 400 (application/json)

        {
           "errors": [
              {
                 "message": "Failed to generate message",
                 "description": "Failed to generate verification email",
                 "code": "1901"
              }
           ]
        }
