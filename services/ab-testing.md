title: A/B Testing
description: A/B Testing of templates.

# Group A/B Testing

<a name="ab-testing-api"></a>

An A/B test is a method of comparing templates against a default template to see how their performance compares.  Users specify a default template and up to twenty template variants to compare, the comparison is based on the user selected metric.  Currently there are two supported modes of audience selection (which recipients receive the variant templates): a fixed number of recipients per variant can be specified, alternatively a percentage of recipients per variant can be specified.  There are two supported modes of behavior selection once the A/B test completes.  In Learning Mode once the test has completed subsequent transmissions will revert to using the default template.  In Bayesian mode the best performing template (the "winner") as determined by a Bayesian algorithm will be used in subsequent transmissions.

#### A/B Test Properties

| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| id | string | The identifier for this A/B test | |
| name | string | A human readable name for this A/B test | |
| status | string | The current state of the test.  Possible values: `scheduled`, `running`, `completed`, `cancelled` | GET only |
| final_template | string | The "winner" of the A/B test (only present if the state is `completed`) | GET only |
| version | integer | The current version number of the test.  The version increments each time the A/B test is modified. | |
| default_template | object | Details for the default template. See [Template Properties](#header-template-properties) | |
| variants | array | Specifies which variants to test, as well as how messages are distributed to each variant. See [Template Properties](#header-template-properties) | |
| metric | string | One of `count_unique_clicked`, `count_unique_rendered` | |
| audience_selection | string | Determines how to distribute messages for templates. Each template will receive either a percent of the total of all messages or a set number of messages determined by the template's sample_size. Options are `percent`, `sample_size` | |
| test_mode | string | Either `bayesian` or `learning` | |
| start_time | string | ISO Date specifying when the test should begin | |
| end_time | string | (Optional) ISO Date specifying when the test should end | |
| total_sample_size | int | (Optional) Total number of messages to send as part of the test | |
| confidence_level | float | (Optional) Specify a confidence level at which point the test should end | Defaults to 0.95 |
| engagement_timeout | int | (Optional) The amount of time, in hours, until the lack of an engagement event is counted as a failure. | Defaults to 24 hours |
| created_at | string | ISO Date of A/B Test Creation | GET only |
| updated_at | string | ISO Date of the last time the A/B test was updated | GET only |

### Template Properties
| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| template_id | string | The template id | |
| sample_size | integer | The number of injections to send using this template | Required if A/B test has `total_sample_size` defined |
| percent | integer | The percent of injections to send using this template | Required if A/B test does not have `total_sample_size` defined |

<div class="alert alert-info"><strong>Note</strong>: Tests will run until one of the following criteria are met: The end_time has passed, messages equal to the total_sample_size have been sent, or the confidence_level (Bayesian mode only) has been reached. In Bayesian mode, reaching the specified confidence_level for a template will cause it to become the "winner". If a test ends and the confidence_level has not been reached, the default template will be considered the "winner". </div>

## A/B Tests [/api/v1/ab-test]

## Create an A/B Test using a percentage for distribution [POST]

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "metric": "count_unique_opened",
          "audience_selection": "percent",
          "start_time": "2018-04-03T22:08:33Z",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "percent": 50
          },
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "percent": 25
            },
            {
              "template_id": "payment_confirmation_variant2",
              "percent": 25
            }
          ]
        }
    ```

+ Response 200 (application/json)

    ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "Variants must have a template_id"}]
    }
    ```


## Create an A/B Test using sample_size for distribution [POST]

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "metric": "count_unique_opened",
          "audience_selection": "sample_size",
          "start_time": "2018-04-03T22:08:33+00:00",
          "test_mode": "learning",
          "total_sample_size": 60000,
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "sample_size": 40000
          },
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "sample_size": 10000
            },
            {
              "template_id": "payment_confirmation_variant2",
              "sample_size": 10000
            }
          ]
        }
    ```

+ Response 200 (application/json)

     ```json
    {
      "results": {
        "id": "payment-confirmation"
      }
    }
    ```

+ Response 400 (application/json)

    ```json
    {
      "errors": [{"message": "Variants must have a template_id"}]
    }
    ```

## List All A/B Tests [GET]

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

  ```json
    {
      "results": [
        {
          "id": "payment-confirmation",
          "name": "Payment Confirmation",
          "version": 2,
          "status": "running",
          "metric": "count_unique_opened",
          "audience_selection": "percent",
          "start_time": "2018-04-03T22:08:33+00:00",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "default_template": {
            "template_id": "default_payment_confirmation_template",
            "percent": 60
          },
          "variants": [
            {
              "template_id": "payment_confirmation_variant1",
              "percent": 10
            },
            {
              "template_id": "payment_confirmation_variant2",
              "percent": 30
            }
          ]
        },
        {
          "id": "password-reset",
          "name": "Password Reset",
          "version": 2,
          "status": "completed",
          "final_template": "password_reset_variant2",
          "metric": "count_unique_clicked",
          "audience_selection": "percent",
          "start_time": "2018-04-03T22:08:33+00:00",
          "test_mode": "bayesian",
          "confidence_level": 0.99,
          "default_template": {
            "template_id": "default_password_reset_template",
            "percent": 70
          },
          "variants": [
            {
              "template_id": "password_reset_variant1",
              "percent": 15
            },
            {
              "template_id": "password_reset_variant2",
              "percent": 15
            }
          ]
        }
      ]
    }
  ```

## A/B Tests Resource [/api/v1/ab-test/{id}{?version}]

## Get an A/B Test [GET]

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID
  + version (optional, integer) ... If passed return information about the specific version of the A/B test. If not specified, return information about the latest version.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

     ```json
     {
     "results": {
        "id": "password-reset",
        "name": "Password Reset",
        "version": 2,
        "status": "scheduled",
        "metric": "count_unique_opened",
        "audience_selection": "sample_size",
        "start_time": "2018-04-03T22:08:33+00:00",
        "test_mode": "bayesian",
        "confidence_level": 0.99,
        "total_sample_size": 60000,
        "default_template": {
          "template_id": "default_password_reset_template",
          "sample_size": 20000
        },
        "variants": [
          {
            "template_id": "password_reset_variant1",
            "sample_size": 20000
          },
          {
            "template_id": "password_reset_variant2",
            "sample_size": 20000
          }
         ]
       }
     }
    ```

 + Response 404 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset does not exist"}]
    }
    ```

## Update an A/B Test [/api/v1/ab-test/{id}]

### Update an A/B Test [PUT]

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID

<div class="alert alert-info"><strong>Note</strong>: Updating an A/B test creates a new version of the test.  This effectively causes the test to restart. Tests in `running` state must be cancelled before updating.</div>

+ Modify an A/B test properties

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json
    + Body
      ```json
      {
        "total_sample_size": 100000,
        "default_template": {
          "template_id": "default_password_reset_template",
          "sample_size": 70000
        },
        "variants": [
          {
            "template_id": "password_reset_variant1",
            "sample_size": 10000
          },
          {
            "template_id": "password_reset_variant2",
            "sample_size": 20000
          }
        ]
      }
      ```


+ Response 200 (application/json)
    ```json
    {
      "results": {
        "version": 2
      }
    }
    ```

+ Response 409 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset is running"}]
    }
    ```

## Delete an A/B Test [DELETE]

+ Request Delete an A/B test

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID

+ Response 200 (application/json)
      ```
      { "results": {} }
      ```

+ Response 404 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset does not exist"}]
    }
    ```


## Cancel an A/B Test [/api/v1/ab-test/{id}/cancel]

### Cancel an A/B Test [POST]

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID

+ Request Cancel an A/B test

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json


+ Response 200 (application/json)

    ```json
    {
      "results": {
        "status": "cancelled"
      }
    }
    ```

+ Response 404 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password-reset does not exist"}]
    }
    ```


## A/B Tests Stat Resource [/api/v1/ab-test/{id}/stats?version=1]

### Get Stats for an A/B Test [GET]


<div class="alert alert-info"><strong>Note</strong>: This only provides very high level summary statistics - the success and failure counts, as well as the confidence level of particular templates being the "winner" based on a Bayesian algorithm approach.  If you need finer granular detail you should use the Message Events API or Event Webhooks</div>

<div class="alert alert-info"><strong>Note</strong>: count_injected may be larger than the sum of count_success and count_failure - this is because there is some delay for allowing engagement events to be processed</div>

+ Parameters

  + id (required, string, `password-reset`) ... A/B Test ID
  + version (optional, integer) ... If passed return information about the specific version of the A/B test.  If not specified, return information about the latest version.

+ Request Get Stats for an A/B Test

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)
      ```
      {
        "results": {
          "templates": [
            {
              "template_id": "password_reset_default",
              "count_injected": 10000,
              "count_success": 7777,
              "count_failure": 1111
              "confidence_level": .99,
            },
            {
              "template_id": "password_reset_variant1",
              "count_injected": 10000,
              "count_success": 5555,
              "count_failure": 3333,
              "confidence_level": .001,
            },
            {
              "template_id": "password_reset_variant1",
              "count_injected": 10000,
              "count_success": 1111,
              "count_failure": 7777,
              "confidence_level": .001
            }
          ]
        }
      }
      ```

+ Response 404 (application/json)

    ```json
    {
      "errors": [{"message": "A/B test password_reset does not exist"}]
    }
    ```
