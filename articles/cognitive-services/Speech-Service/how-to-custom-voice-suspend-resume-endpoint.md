# How to resume / suspend endpoint via REST API

The endpoint Suspend and Resume functions allows you to flexibly control the endpoint hosting status and save the hosting costs. The charging is stopped once the endpoint is suspended, meanwhile it cannot be used to synthesize speech until you resume it successfully.

> [!NOTE]
> The Suspend or Resume operation will take a while to complete, the Suspend time should short and the Resume time should be similar as the Deploy operation.

This how-to topic will show you how to suspend or resume a Custom Voice endpoint via REST API.

> [!Tip]
> The endpoint Suspend and Resume functions have been supported on [Speech Studio portal](https://aka.ms/custom-voice-portal) too.

## Prerequisites

An existing endpoint you want to suspend or resume, you will need to prepare:

- The identifier of the endpoint.

- The Azure region the endpoint is associated with.

- The subscription key the endpoint is associated with.

They can be found in the Sample code section on the endpoint detail page at [Speech Studio portal](https://aka.ms/custom-voice-portal) as below:

![Endpoint parameter for rest API](media/custom-voice/endpoint-parameter-for-rest-api.png)

## Suspend endpoint

Suspend the endpoint identified by the given ID, which applies to the endpoint in `Succeeded` status.

1. Follow the sample request below to call the API, and you will receive the response with `HTTP 202 Status code`.

   Replace the request parameters refer to the [Request Parameter](#request-parameter).

2. Follow the [Get endpoint](#get-endpoint) step to track the operation progress, you can poll the get endpoint API in a loop until the status becomes `Disabled`, the status property will change from `Succeeded` status, to `Disabling`, and finally to `Disabled`.

### Sample Request

HTTP sample

```HTTP
POST api/texttospeech/v3.0/endpoints/<Endpoint_ID>/suspend HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: <REGION_IDENTIFIER>.customvoice.api.speech.microsoft.com
Content-Type: application/json
Content-Length: 0
```

cURL sample - Run with command-line tool in Linux (and in the Windows Subsystem for Linux).

```Console
curl -v -X POST "https://<REGION_IDENTIFIER>.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/endpoints/<Endpoint_ID>/suspend" -H "Ocp-Apim-Subscription-Key: <YOUR_SUBSCRIPTION_KEY >" -H "content-type: application/json" -H "content-length: 0"
```

### Sample Response

Status code: 202 Accepted

See [Response Header](response-header) for details.

## Resume endpoint

Resume the endpoint identified by the given ID, which applies to the endpoint in `Disabled` status.

1. Follow the sample request below to call the API, and you will receive the response with `HTTP 202 Status code`.

   Replace the request parameters refer to the [Request Parameter](#request-parameter).

2. Follow the [Get endpoint](#get-endpoint) step to track the operation progress, you can poll the API in a loop until the status becomes `Succeeded` or `Disabled`, the status property will change from `Disabled` status, to `Running`, and finally to `Succeeded` or `Disabled` if failed.

### Sample Request

HTTP sample

```HTTP
POST api/texttospeech/v3.0/endpoints/<Endpoint_ID>/resume HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: <REGION_IDENTIFIER>.customvoice.api.speech.microsoft.com
Content-Type: application/json
Content-Length: 0
```

cURL sample - Run with command-line tool in Linux (and in the Windows Subsystem for Linux).

```Console
curl -v -X POST "https://<REGION_IDENTIFIER>.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/endpoints/<Endpoint_ID>/resume" -H "Ocp-Apim-Subscription-Key: <YOUR_SUBSCRIPTION_KEY >" -H "content-type: application/json" -H "content-length: 0"
```

### Sample Response

Status code: 202 Accepted

See [Response Header](response-header) for details.

## Get endpoint

Get the endpoint identified by the given ID, this API allows you to query the details of the endpoint.

1. Follow the sample request below to call the API.

   Replace the request parameters refer to the [Request Parameter](#request-parameter).

2. Check the status property in response payload to track the progress for Suspend or Resume operation.

The definition of status property:
| Status | Description |
| ------------- | ------------------------------------------------------------ |
| `NotStarted` | The endpoint is waiting for processing for Deploy, it's not ready to synthesize speech. |
| `Running` | The endpoint is in processing for Deploy or Resume, it's not ready to synthesize speech. |
| `Succeeded` | The endpoint is succeeded to Deploy or Resume, it's ready to synthesize speech. |
| `Failed` | The endpoint is in processing for Suspend. |
| `Disabling` | The endpoint is waiting for processing for Deploy, it's not ready to synthesize speech. |
| `Disabled` | The endpoint is succeeded to Suspend or failed to Resume. |

> [!Tip]
> If the status goes to Failed or Disabled for Resume, you can check the `properties.error` for the detailed error message.

### Sample Request

HTTP sample

```HTTP
GET api/texttospeech/v3.0/endpoints/<Endpoint_ID> HTTP/1.1
Ocp-Apim-Subscription-Key: YOUR_SUBSCRIPTION_KEY
Host: <REGION_IDENTIFIER>.customvoice.api.speech.microsoft.com
```

cURL sample - Run with command-line tool in Linux (and in the Windows Subsystem for Linux).

```Console
curl -v -X GET "https://<REGION_IDENTIFIER>.customvoice.api.speech.microsoft.com/api/texttospeech/v3.0/endpoints/<Endpoint_ID>" -H "Ocp-Apim-Subscription-Key: <YOUR_SUBSCRIPTION_KEY >"
```

### Sample Response

Status code: 200 OK

```json
{
  "model": {
    "id": "a92aa4b5-30f5-40db-820c-d2d57353de44"
  },
  "project": {
    "id": "ffc87aba-9f5f-4bfa-9923-b98186591a79"
  },
  "properties": {},
  "status": "Succeeded",
  "lastActionDateTime": "2019-01-07T11:36:07Z",
  "id": "e7ffdf12-17c7-4421-9428-a7235931a653",
  "createdDateTime": "2019-01-07T11:34:12Z",
  "locale": "en-US",
  "name": "Voice endpoint",
  "description": "Example for voice endpoint"
}
```

## Request Parameter

Replace the parameters with proper data you found at [Prerequisites](#prerequisites) step.

| Name                        | In     | Required | Type   | Description                                                                    |
| --------------------------- | ------ | -------- | ------ | ------------------------------------------------------------------------------ |
| `Region`                    | Path   | `True`   | string | <REGION_IDENTIFIER> - The Azure region the endpoint is associated with.        |
| `Endpoint_ID`               | Path   | `True`   | string | <Endpoint_ID> - The identifier of the endpoint.                                |
| `Ocp-Apim-Subscription-Key` | Header | `True`   | string | <YOUR_SUBSCRIPTION_KEY > The subscription key the endpoint is associated with. |

## Response Header

Status code: 202 Accepted

| Name          | Type   | Description                                                                      |
| ------------- | ------ | -------------------------------------------------------------------------------- |
| `Location`    | string | The location of the endpoint which can be used as the full URL to get endpoint . |
| `Retry-After` | string | The total seconds of recommended interval to retry to get endpoint status.       |
