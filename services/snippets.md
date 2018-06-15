title: Snippets
description: Manage reusable content snippets that can be referenced within inline content and stored templates. 

# Group Snippets

The Snippets API provides the means to manage your reusable content snippets which can be referenced within inline content and stored templates.
Refer to the [Substitutions Reference](https://developers.sparkpost.com/api/substitutions-reference.html) page for details of the substitution macro call
which allows a snippet to be imported into a specific location in the html and/or text email content.

## Snippet Attributes

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|id    | string  |Short, unique, alphanumeric ID used to reference the snippet   | yes |After a snippet has been created, this property cannot be changed.  Maximum length - 64 bytes   |
|content              |JSON  |HTML and plain text content of the snippet  |  yes  |  For a full description, see the [Content Attributes](#header-content-attributes). |
|shared_with_subaccounts | boolean | Whether this snippet can be used by subaccounts | no | Defaults to `false`.  Only available to snippets belonging to a master account.|

### Content Attributes

Content for a snippet is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|html    |string  |HTML content which will be used when the snippet is referenced from the HTML part of a stored template or inline content.  |no.  |Expected in the UTF-8 charset with no `Content-Transfer-Encoding` applied.  Substitution syntax is supported with the exception of the render_dynamic_content() and render_snippet() macro calls.  In other words, snippets may not reference other snippets. Maximum length - 100KB |
|text    |string  |Plain text content which will be used when the snippet is referenced from the plain text part of a stored template or inline content.  |no.  |Expected in the UTF-8 charset with no `Content-Transfer-Encoding` applied.  Substitution syntax is supported with the exception of the render_dynamic_content() and render_snippet() macro calls.  In other words, snippets may not reference other snippets. Maximum length - 100KB|

## Create and List [/snippets]

### Create a Snippet [POST]

Create a snippet by providing a **snippet object** as the POST request body.

The `id` and `content` fields are required, where content must contain at least one of the 'html' or 'text' fields.

+ Request Create Basic Snippet (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
            "id" : "header",
            "shared_with_subaccounts": false,

            "content": {
                "html": "<b>Header snippet data for {{name}}</b>"
            }
        }
        ```

+ Response 200 (application/json)

        {
          "results": {
            "id": "header"
          }
        }

+ Response 400 (application/json)

        {
          "errors" : [
            {
              "description" : "Subaccounts cannot set the shared_with_subaccounts flag",
              "code" : "1200",
              "message" : "invalid params"
            }
          ]
        }

+ Response 422 (application/json)

        {
          "errors" : [
            {
              "part" : "text",
              "description" : "Error while compiling part text: line 4: syntax error near 'age'",
              "line" : 4,
              "code" : "3000",
              "message" : "substitution language syntax error in snippet content"
            }
          ]
        }


### List all Snippets [GET /snippets]

Lists each snippet in your account.

Each snippet object in the list will have the following field:
- id: Unique snippet ID.

Additionally, snippets owned by the Master account will have the following field:
- shared_with_subaccounts: Whether the snippet is shared with subaccounts.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results" : [
            {
              "id" : "header",
              "shared_with_subaccount" : true
            },
            {
              "id" : "footer",
              "shared_with_subaccount" : true
            }
          ]
        }

### List Snippets selectively [GET /snippets{?shared_with_subaccounts}]

Lists each shared snippet to which you have access.

Each snippet object in the list will have the following field:
- id: Unique snippet ID.

Additionally, snippets owned by the Master account will have the following field:
- shared_with_subaccounts: Whether the snippet is shared with subaccounts.

+ Parameters
    + shared_with_subaccounts (optional, boolean, `true`) ...If true, returns only shared snippets. If false, returns only non-shared snippets.  When not provided, returns both shared and non-shared snippets.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results" : [
            {
              "id" : "header",
              "shared_with_subaccount" : true
            },
            {
              "id" : "footer",
              "shared_with_subaccount" : true
            }
          ]
        }


## Retrieve [/snippets/{id}]

### Retrieve a Snippet [GET]

Retrieve a single snippet by specifying its ID in the URI path.

For a master account owned snippet **only**, the result will include the `shared_with_subaccounts` field reflecting the snippet's shared status.

+ Parameters
    + id (required, string, `ourfooter`) ... ID of the snippet

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results" : {
            "id" : "ourfooter",
            "content": {
              "html" : "<b>Our standard footer</b>",
              "text" : "Our standard footer"
            }
          }
        }

## Update [/snippets/{id}]

### Update a Snippet [PUT]

Update an existing snippet by specifying its ID in the URI path and a **snippet object** as the PUT request body.

The `id` field may not be updated.

If a content object is provided in the update request, it must
contain all relevant content fields whether they are being changed or not.
The new content will completely overwrite the existing content.

+ Parameters
    + id (required, string, `ourfooter`) ... ID of the snippet

+ Request Update (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
       {
         "content": {
           "html" : "<b>Our updated footer</b>",
           "text" : "Our updated footer"
          },
          "shared_with_subaccounts" : false
        }
        ```

+ Response 200

## Delete [/snippets/{id}]

### Delete a Snippet [DELETE]

Delete a snippet by specifying its ID in the URI path.

+ Parameters
    + id (required, string, `ourfooter`) ... ID of the snippet

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 204 (application/json)

+ Response 404 (application/json)

    + Body

        ```
        {
        "errors": [
          {
            "message": "resource not found",
            "code": "1600",
            "description": "Snippet does not exist"
          }
        ]
        }
        ```
