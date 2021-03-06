# //staq REST API documentation V1.0

[//staq](http://staq.io) is a platforms to mange and operate service-based games.

Version 1 of the API is limited to event tracking and IAP purchase server verification.

The API is a [REST API](http://en.wikipedia.org/wiki/Representational_State_Transfer "RESTful")
and uses an AppSecret specified in the http header for authenticating request.

You can signup at [http://staq.io](http://staq.io) or login at [https://dashboard.staq.io](https://dashboard.staq.io).

AppSecret and AppId are visible in the dashboard by clicking on **User => Show Api Keys**.
Input and output format is [JSON](http://json.org/ "JSON").

***

## Endpoint

**<code>Base url</code>**: https://api.staq.io/v1

# API methods

- **[<code>POST</code> /apps/{appId}/auth/device/{deviceId}](#device-authorization)**
- **[<code>POST</code> /apps/{appId}/users/{userId}/iap/apple](#ios-iap-verification)**
- **[<code>POST</code> /apps/{appId}/users/{userId}/iap/googleplay](#google-play-iap-verification)**
- **[<code>POST</code> /apps/{appId}/users/{userId}/events](#custom-events)**

## Device Authorization

<code>POST /apps/{appId}/auth/device</code>

### Description

This resource allows you to authorize an anonymous user. You need to call this method before consuming any other resource from the staq API.

This method will log the following events:

Logged events:

- <code>server_INSTALL</code> - New installation event
- <code>server_SESSION_START</code> - New session event

### Request parameters

**Url parameters:**

<table>
	<tr>
		<td><code>appId</code></td>
		<td>An application id</td>
	</tr>
</table>
 
**Json Parameters:**

<table>
	<tr>
		<td><code>deviceId</code></td>
		<td>An arbitrary device id or <a href="http://en.wikipedia.org/wiki/Universally_unique_identifier">UUID</a></td>
	</tr>
	<tr>
		<td><code>deviceInfo</code></td>
		<td>An arbitray JSON object contains platform and/or other custom information that you want to associate with the user.</td>
	</tr>
</table>

### Response

<table>
	<tr>
		<td><code>appId</code></td>
		<td>Your application id</td>
	</tr>
	<tr>
		<td><code>uid</code></td>
		<td>An id that staq assign to the current user. This is required as input parameter by other api resources.</td>
	</tr>
	<tr>
		<td><code>sid</code></td>
		<td>An id associated with the current session.</td>
	</tr>
	<tr>
		<td><code>token</code></td>
		<td>An authorization token associated with the current user. This is required as input parameter by other api resources.</td>
	</tr>
</table>

### Example

```http
Request:
POST https://api.staq.io/v1/apps/app-000001/auth/device HTTP/1.1
Host: staq.io
Content-Type: application/json; charset=utf-8

{
	"deviceId": "my device ID",
	"deviceInfo":
	{
		"env-variable-1": "value1", 
		"other-env-variable": "value2"   
	}
}

Response:
200 (OK)
Content-Type: application/json

{
   "appId":"your-app-id",
   "uid":"staq-user-id",
   "sid":"session-id",
   "token":"auth-token"
}
```

## iOS IAP Verification

<code>POST /apps/{appId}/users/{uid}/iap/apple</code>

### Description

This resource allows you to verify an iOS in-app purchase.

This method will log the following events:

Logged events:

- <code>server_IAP_COMPLETED</code> - Completed IAP event

### Request parameters

**HTTP Header parameters:**

<table>
	<tr>
		<td><code>Auth-Token</code></td>
		<td>An authorization token associated with the current user</td>
	</tr>
</table>

**Url parameters:**

<table>
	<tr>
		<td><code>appId</code></td>
		<td>Application id</td>
	</tr>
	<tr>
		<td><code>uid</code></td>
		<td>User id provided by the authroization method of the staq API.</td>
	</tr>
</table>
 
**Json Parameters:**

<table>
	<tr>
		<td><code>sid</code></td>
		<td>Session id provided by the authroization call of the staq API.</td>
	</tr>
	
	<tr>
		<td><code>receipt</code></td>
		<td>An <a href="http://developer.apple.com/library/ios/#documentation/NetworkingInternet/Conceptual/StoreKitGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008267-CH1-SW1">iOS receipt to validate</a></td>
	</tr>
	<tr>
		<td><code>priceUsd</code></td>
		<td>Numeric value of the price of the item expressed in USD. <em>ex: 9.99</em></td>
	</tr>
	<tr>
		<td><code>status</code></td>
		<td>Status of the current IAP. Possible values are: <code>processing</code> and <code>completed</code></td>
	</tr>
	<tr>
		<td><code>comment</code></td>
		<td>A comment string to attach to the current transaction</td>
	</tr>
	<tr>
		<td><code>metadata</code></td>
		<td>An arbitray JSON blob contains platform and/or other custom information that you want to associate with the user.</td>
	</tr>
</table>


### Response

<table>
	<tr>
		<td><code>isValid</code></td>
		<td>Boolean value. True if this is a valid receipt.</td>
	</tr>
	<tr>
		<td><code>isDuplicate</code></td>
		<td>Boolean value. True if the receipt has been already validated.</td>
	</tr>
</table>

### Example

```http
Request:

POST /apps/app-000001/users/uid-00001/iap/apple HTTP/1.1
Host: example.org
Content-Type: application/json; charset=utf-8
Auth-Token: your-auth-token

{ 
    "sid": "session-id",
    "receipt" : "ew0KCSJzaWduYXR1cmUiID0gIkFoRi94VkpCcmFIUWQ1RzNpSFlObFBQcFhwRUh4U2R0bEIvUW9oTGtLTlRWcUpjN3B6OWQ3UjZ6QUI4WXM2S0UzSUhCR3BxU1NuNTBVWnJrVEVLRFlkM1cxTHdzRTNiU1RJMHlobk9HRm84Sk56R0p6R0lDSDU2OFlKckg5V3NWR0hRdnJTTjg1bVVVeEROWGlnOVdRMmVCdVhpUnE5VWZVNS9YL3NxZ1hnbkJBQUFEVnpDQ0ExTXdnZ0k3b0FNQ0FRSUNDR1VVa1UzWldBUzFNQTBHQ1NxR1NJYjNEUUVCQlFVQU1IOHhDekFKQmdOVkJBWVRBbFZUTVJNd0VRWURWUVFLREFwQmNIQnNaU0JKYm1NdU1TWXdKQVlEVlFRTERCMUJjSEJzWlNCRFpYSjBhV1pwWTJGMGFXOXVJRUYxZEdodmNtbDBlVEV6TURFR0ExVUVBd3dxUVhCd2JHVWdhVlIxYm1WeklGTjBiM0psSUVObGNuUnBabWxqWVhScGIyNGdRWFYwYUc5eWFYUjVNQjRYRFRBNU1EWXhOVEl5TURVMU5sb1hEVEUwTURZeE5ESXlNRFUxTmxvd1pERWpNQ0VHQTFVRUF3d2FVSFZ5WTJoaGMyVlNaV05sYVhCMFEyVnlkR2xtYVdOaGRHVXhHekFaQmdOVkJBc01Fa0Z3Y0d4bElHbFVkVzVsY3lCVGRHOXlaVEVUTUJFR0ExVUVDZ3dLUVhCd2JHVWdTVzVqTGpFTE1Ba0dBMVVFQmhNQ1ZWTXdnWjh3RFFZSktvWklodmNOQVFFQkJRQURnWTBBTUlHSkFvR0JBTXJSakYyY3Q0SXJTZGlUQ2hhSTBnOHB3di9jbUhzOHAvUndWL3J0LzkxWEtWaE5sNFhJQmltS2pRUU5mZ0hzRHM2eWp1KytEcktKRTd1S3NwaE1kZEtZZkZFNXJHWHNBZEJFakJ3Ukl4ZXhUZXZ4M0hMRUZHQXQxbW9LeDUwOWRoeHRpSWREZ0p2MllhVnM0OUIwdUp2TmR5NlNNcU5OTEhzREx6RFM5b1pIQWdNQkFBR2pjakJ3TUF3R0ExVWRFd0VCL3dRQ01BQXdId1lEVlIwakJCZ3dGb0FVTmgzbzRwMkMwZ0VZdFRKckR0ZERDNUZZUXpvd0RnWURWUjBQQVFIL0JBUURBZ2VBTUIwR0ExVWREZ1FXQkJTcGc0UHlHVWpGUGhKWENCVE16YU4rbVY4azlUQVFCZ29xaGtpRzkyTmtCZ1VCQkFJRkFEQU5CZ2txaGtpRzl3MEJBUVVGQUFPQ0FRRUFFYVNiUGp0bU40Qy9JQjNRRXBLMzJSeGFjQ0RYZFZYQWVWUmVTNUZhWnhjK3Q4OHBRUDkzQmlBeHZkVy8zZVRTTUdZNUZiZUFZTDNldHFQNWdtOHdyRm9qWDBpa3lWUlN0USsvQVEwS0VqdHFCMDdrTHM5UVVlOGN6UjhVR2ZkTTFFdW1WL1VndkRkNE53Tll4TFFNZzRXVFFmZ2tRUVZ5OEdYWndWSGdiRS9VQzZZNzA1M3BHWEJrNTFOUE0zd294aGQzZ1NSTHZYaitsb0hzU3RjVEVxZTlwQkRwbUc1K3NrNHR3K0dLM0dNZUVONS8rZTFRVDlucC9LbDFuaithQnc3QzB4c3kwYkZuYUFkMWNTUzZ4ZG9yeS9DVXZNNmd0S3Ntbk9PZHFUZXNicDBiczhzbjZXcXMwQzlkZ2N4Ukh1T01aMnRtOG5wTFVtN2FyZ09TelE9PSI7DQoJInB1cmNoYXNlLWluZm8iID0gImV3b0pJbTl5YVdkcGJtRnNMWEIxY21Ob1lYTmxMV1JoZEdVdGNITjBJaUE5SUNJeU1ERXlMVEV4TFRJd0lERXhPakV3T2pBMklFRnRaWEpwWTJFdlRHOXpYMEZ1WjJWc1pYTWlPd29KSW5WdWFYRjFaUzFwWkdWdWRHbG1hV1Z5SWlBOUlDSXdNREF3WWpBeU1UZzRNVGdpT3dvSkltOXlhV2RwYm1Gc0xYUnlZVzV6WVdOMGFXOXVMV2xrSWlBOUlDSXhNREF3TURBd01EVTRPVEV5TkRFd0lqc0tDU0ppZG5KeklpQTlJQ0l4TGpBaU93b0pJblJ5WVc1ellXTjBhVzl1TFdsa0lpQTlJQ0l4TURBd01EQXdNRFU0T1RFMU5EazNJanNLQ1NKeGRXRnVkR2wwZVNJZ1BTQWlNU0k3Q2draWIzSnBaMmx1WVd3dGNIVnlZMmhoYzJVdFpHRjBaUzF0Y3lJZ1BTQWlNVE0xTXpRek9EWXdOakF3TUNJN0Nna2ljSEp2WkhWamRDMXBaQ0lnUFNBaU1TSTdDZ2tpYVhSbGJTMXBaQ0lnUFNBaU5UZ3dNamN4TlRRMklqc0tDU0ppYVdRaUlEMGdJbWx2TG5OMFlYRXVhR1ZzYkc5cFlYQWlPd29KSW5CMWNtTm9ZWE5sTFdSaGRHVXRiWE1pSUQwZ0lqRXpOVE0wTXprNU56WXdNREFpT3dvSkluQjFjbU5vWVhObExXUmhkR1VpSUQwZ0lqSXdNVEl0TVRFdE1qQWdNVGs2TXpJNk5UWWdSWFJqTDBkTlZDSTdDZ2tpY0hWeVkyaGhjMlV0WkdGMFpTMXdjM1FpSUQwZ0lqSXdNVEl0TVRFdE1qQWdNVEU2TXpJNk5UWWdRVzFsY21sallTOU1iM05mUVc1blpXeGxjeUk3Q2draWIzSnBaMmx1WVd3dGNIVnlZMmhoYzJVdFpHRjBaU0lnUFNBaU1qQXhNaTB4TVMweU1DQXhPVG94TURvd05pQkZkR012UjAxVUlqc0tmUT09IjsNCgkiZW52aXJvbm1lbnQiID0gIlNhbmRib3giOw0KCSJwb2QiID0gIjEwMCI7DQoJInNpZ25pbmctc3RhdHVzIiA9ICIwIjsNCn0=",
    "priceUsd" : 9.99,
    "status" : "completed",
    "comment" : "this is a comment",
    "metadata": {
        "env-variable-1": "value1", 
        "other-env-variable": "value2"   
    } 
}

Response:
200 (OK)
Content-Type: application/json

{
   "isValid":true,
   "isDuplicated":false
}
```

## Google Play IAP Verification

<code>POST /apps/{appId}/users/{uid}/iap/googleplay</code>

### Description

This resource allows you to verify a Google Play in-app purchase.

This method will log the following events:

Logged events:

- <code>server_IAP_COMPLETED</code> - Completed IAP event

### Request parameters

**HTTP Header parameters:**

<table>
	<tr>
		<td><code>Auth-Token</code></td>
		<td>An authorization token associated with the current user</td>
	</tr>
</table>

**Url parameters:**

<table>
	<tr>
		<td><code>appId</code></td>
		<td>Application id</td>
	</tr>
	<tr>
		<td><code>uid</code></td>
		<td>User id provided by the authroization method of the staq API.</td>
	</tr>
</table>
 
**Json Parameters:**

<table>
	<tr>
		<td><code>sid</code></td>
		<td>Session id provided by the authroization call of the staq API.</td>
	</tr>
	
	<tr>
		<td><code>receipt</code></td>
		<td>The <strong>Base 64</strong> encoded data that is mapped to the INAPP_PURCHASE_DATA key in the response Intent from the <a href="http://developer.android.com/google/play/billing/billing_integrate.html">Google Play In-App Billing API</a>.</td>
	</tr>
	<tr>
		<td><code>signature</code></td>
		<td>The <strong>Base 64</strong> encoded data that is mapped to the INAPP_DATA_SIGNATURE key in the response Intent from the <a href="http://developer.android.com/google/play/billing/billing_integrate.html">Google Play In-App Billing API</a>.</td>
	</tr>
	<tr>
		<td><code>status</code></td>
		<td>Status of the current IAP. Possible values are: <code>processing</code> and <code>completed</code></td>
	</tr>
	<tr>
		<td><code>priceUsd</code></td>
		<td>Numeric value of the price of the item expressed in USD. <em>ex: 9.99</em></td>
	</tr>
	
	<tr>
		<td><code>comment</code></td>
		<td>A comment string to attach to the current transaction</td>
	</tr>
	<tr>
		<td><code>metadata</code></td>
		<td>An arbitray JSON blob contains platform and/or other custom information that you want to associate with the user.</td>
	</tr>
</table>


### Response

<table>
	<tr>
		<td><code>isValid</code></td>
		<td>Boolean value. True if this is a valid receipt.</td>
	</tr>
	<tr>
		<td><code>isDuplicate</code></td>
		<td>Boolean value. True if the receipt has been already validated.</td>
	</tr>
</table>

### Example

```http 
Request:

POST /apps/app-000001/users/uid-00001/iap/googleplay HTTP/1.1
Host: example.org
Content-Type: application/json; charset=utf-8
Auth-Token: your-auth-token

{ 
    "sid": "session-id",
    "receipt" : "BASE64-OF-THE-PURCHASE-DATA",
    "signature" : "BASE64-OF-THE-SIGNATURE-DATA",
    "status" : "completed",
    "priceUsd" : 9.99,
    "comment" : "this is a comment",
    "metadata": {
        "env-variable-1": "value1", 
        "other-env-variable": "value2"   
    } 
}

Response:
200 (OK)
Content-Type: application/json

{
   "isValid":true,
   "isDuplicated":false
}
```

## Custom Events

<code>POST /apps/{appId}/users/{uid}/events</code>

### Description

This resource allows you to post an array of custom events.


### Request parameters

**HTTP Header parameters:**

<table>
	<tr>
		<td><code>Auth-Token</code></td>
		<td>An authorization token associated with the current user.</td>
	</tr>
</table>

**Url parameters:**

<table>
	<tr>
		<td><code>appId</code></td>
		<td>Application id</td>
	</tr>
	<tr>
		<td><code>uid</code></td>
		<td>User id provided by the authroization call of the staq API.</td>
	</tr>
</table>
 
**Json Parameters:**

<table>
	<tr>
		<td><code>sid</code></td>
		<td>Session id provided by the authroization method of the staq API.</td>
	</tr>
	<tr>
		<td><code>events</code></td>
		<td>
			Array of:
			<table>
				<tr>
					<td><code>timestamp</code></td>
					<td>A valid JSON timestamp</td>
				</tr>
				<tr>
					<td><code>event</event></td>
					<td>Event name</td>
				</tr>
				<tr>
					<td><code>value</code></td>
					<td>Event value</td>
				</tr>
				<tr>
					<td>`metadata`</td>
					<td>An arbitray JSON blob contains platform and/or other custom information that you want to associate with the user.</td>
				</tr>
			</table>
		<td>
	</tr>
</table>


### Response

A string array of event id.

### Example

```http
Request:
POST /apps/app-000001/users/uid-00001/events HTTP/1.1
Host: example.org
Content-Type: application/json; charset=utf-8
Auth-Token: your-auth-token

{ 
    "sid": "session-id",
    "events": [{
	"timestamp" : "\/Date(1330848000000-0800)\/",
	"event" : "my-custom-event",
	"value" : "my-custom-value",
	"metadata" : {
		"env-variable-1": "value1", 
		"other-env-variable": "value2"
	}
    }]
}

Response:
200 (OK)
Content-Type: application/json

["eventid-000001"]
```

