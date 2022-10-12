# Google Pay Payment Request Object
This section lists all available properties that are available for use with TEConnect.
Many of the properties correspond directly to Google's documentation on the payment request object ([found here](https://developers.google.com/pay/api/web/reference/request-objects#PaymentDataRequest)).  Some of the properties (such as ```tokenizationSpecification```) is built by TEConnect, and as such is not read as an input.

Examples for simple implementations can be found here //TODO: link to TecPaymentRequestREADME.
An example for the most complex object allowed can be found below.

For more information on each property - please see [Google's Documentation](https://developers.google.com/pay/api/web/reference/request-objects#PaymentDataRequest).


| Property Name | Input Type | Required | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|
| apiVersion | ```number``` | :x: | 2 | |
| apiVersionMinor | ```number``` | :x: | 0 | |
| allowedCardNetworks | ```Array``` of ```string``` | :heavy_check_mark: |  | |
