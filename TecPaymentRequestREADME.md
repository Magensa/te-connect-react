# TEConnect Payment Request (React)
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-react.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-react "@magensa/te-connect-react npm.js")  
React components for use with Apple Pay/Google Pay via Token Exchange Connect

# Payment Request Platforms
TEConnect currently offers two payment request platforms to create payment tokens: [Apple Pay](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md), and [Google Pay](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md).  The payment tokens are processed via [Magensa's Payment Protection Gateway](https://svc1.magensa.net/MPPGv4/MPPGv4Service.svc) (MPPG). To that end - there are many configurable options exposed when using [Apple Pay](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md), and [Google Pay](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md) via TEConnect.  This document will focus on a very simple implementation using both platforms. If you are interested in a more customized experience for your users, using [Apple Pay](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md), and/or [Google Pay](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md) - check out the [TecApplePayREADME.md](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md), and/or [TecGooglePayREADME](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md) which will cover more specific configuration for each - as well as links to Apple/Google's documentation, respectively.

# Getting Started
```
npm install @magensa/te-connect @magensa/te-connect-react
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-react
```

If you would prefer to let the code speak, below we have two [example implementations](#Example-Implementation)

# Step-by-step
1. The first step is to create a TEConnect instance, and feed that instance to the wrapper around your application.
    - The first parameter is your public key for [TEConnect manual entry](https://github.com/Magensa/te-connect-react/blob/master/README.md#Manual-Card-Entry) (```CardEntry``` component).
    - The second parameter is the [```options``` object](https://github.com/Magensa/te-connect-react#teconnect-options)
         - For the ```tecPaymentRequest``` options: 
            - supply your ```appleMerchantId``` to enable Apple Pay.
                - ```appleMerchantId``` is obtained after the on-boarding process with Magensa™ is completed. [Please reach out to Magensa™ support team for more info](https://www.magensa.net/support.html)
            - supply your ```googleMerchantId``` to enable Google Pay.
                - ```googleMerchantId``` is obtained after the [on-boarding process with Google](https://pay.google.com/business/console) is completed.
                - For Google Pay capabilities - ```googleMerchantId```, ```googleMerchantName```, and ```gatewayId``` are needed.
                    - ```gatewayId``` is obtained after the on-boarding process with Magensa™ is completed. [Please reach out to Magensa™ support team for more info](https://www.magensa.net/support.html)
                    - ```googleMerchantId``` and ```googleMerchantName``` are obtained using [Google Pay and Wallet Console](https://pay.google.com/business/console). [More info on MerchantInfo here](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo). Be aware that the name and id are fed to TEConnect seperately - which differs from the direct integration [Google Pay documentation](https://developers.google.com/pay/api/web/reference/request-objects#MerchantInfo).

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { TEConnect } from '@magensa/te-connect-react';
import { createTEConnect } from '@magensa/te-connect';

import { ExampleApp } from './exampleApp';


const teConnectInstance = createTEConnect("__publicKeyGoesHere__", { 
    tecPaymentRequest: { 
        appleMerchantId: "__tecAppleMerchantId__",
        googleMerchantId: "__googleMerchantId__" 
    }
});

const App = () => (
    <TEConnect teConnect={ teConnectInstance }>
        <ExampleApp />
    </TEConnect>
);

ReactDOM.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>,
    document.getElementById('root')
);
```

2. Build a [```paymentRequestObject```](#Payment-Request-Object) to define the Payment Request Form.
    - [Apple's Payment Request Object Structure](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Payment-Request-Object), [Google's Payment Request Object Structure](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md#Payment-Request-Object).
    - Once you have built your [payment request object](#Payment-Request-Object). Supply that object to the  [```createPaymentRequestInterface```](#Create-Payment-Request-Interface) function.
        - Note that if you are using multiple platforms (i.e. both Apple and Google) you must seperate the payment request objects by platform (i.e. ```{ googlePay: {...}, applePay: {...}}```).
        - This function will return the [TecPaymentRequestsInterface](#Token-Exchange-Connect-Payment-Request-Interface).
    - It's recommended to create the interface in a ```useEffect``` hook - to ensure the function is available when it's called.
    - Call the [```canMakePayments```](#canMakePayments) function right away, and await the result to determine if the user is capable of using Apple Pay, Google Pay, both, or none.
    - Once the interface has been instantiated, and the [```CanMakePaymentsResult```](#CanMakePaymentsResult) yeilds a positive result - feed the interface to the ```TecPaymentRequestButtons``` component via the ```paymentRequestInterface``` prop (as demonstrated below). 
        - It is recommmended to wrap the ```TecPaymentRequestButtons``` component in a conditional render (as demonstrated below) or ternary, to avoid undesired behavior.

```javascript
import React, { useState } from 'react';
import ReactDOM from 'react-dom';
import { TEConnect, TecPaymentRequestButtons, useTecPaymentsRequest } from '@magensa/te-connect-react';

import { exampleApplePaymentRequestObject, exampleGooglePaymentRequestObject } from '../consts';

const paymentRequestObject = {
    applePay: exampleApplePaymentRequestObject,
    googlePay: exampleGooglePaymentRequestObject
}

export const ExampleApp = () => {
    const [ prInterface, setPrInterface ] = useState(null);
    const createPaymentRequestInterface = useTecPaymentsRequest();

    useEffect(() => {
        //Check to ensure creation function is available
        if (typeof(createPaymentRequestInterface) === 'function') {
            const tecPrInterface = createPaymentRequestInterface(paymentRequestObject);

            tecPrInterface.canMakePayments().then( availablePrMethods => {
                if (availablePrMethods)
                    setPrInterface(tecPrInterface);
            });
        }
    }, [createPaymentRequestInterface]);
    
    return (
        <div style={{ height: '100%' }}>
            { prInterface && <TecPaymentRequestButtons paymentRequestInterface={ prInterface } /> }
        </div>
    );
}
```
3. Finally - when a user submits the payment request form - an event is emitted. Listen to the [```confirm-token```](#confirm-token-Event) event to inspect the result (as demonstrated below).
    - The ```completePayment``` function is a property of the ```confirm-token``` event _for Apple Pay only_. If the function exists - this function must be called within 30 seconds of the event firing - otherwise the Apple Pay form will timeout.
```javascript
import React, { useState } from 'react';
import { TEConnect, TecPaymentRequestButtons, useTecPaymentsRequest } from '@magensa/te-connect-react';
import { exampleApplePaymentRequestObject, exampleGooglePaymentRequestObject } from '../consts';

const paymentRequestObject = {
    applePay: exampleApplePaymentRequestObject,
    googlePay: exampleGooglePaymentRequestObject
}


export const ExampleApp = () => {
    const [ prInterface, setPrInterface ] = useState(null);
    const createPaymentRequestInterface = useTecPaymentsRequest();

    useEffect(() => {
        //Check to ensure creation function is available
        if (typeof(createPaymentRequestInterface) === 'function') {
            const tecPrInterface = createPaymentRequestInterface(paymentRequestObject);

            tecPrInterface.canMakePayments().then( availablePrMethods => {
                if (availablePrMethods)
                    setPrInterface(tecPrInterface);
            });

            tecPrInterface.listenFor('confirm-token', (tokenResp) => {
                const { tokenDetails, completePayment, error } = tokenResp;

                if (error) {
                    //Unsuccessful - check message.
                    if (typeof(completePayment) === 'function')
                        completePayment('failure');
                }
                else {
                    //Successful
                    //Pass `tokenDetails.token` directly to MPPG for processing.
                    //if Apple Pay - close user's payment request form with payment success notification.
                    if (typeof(completePayment) === 'function')
                        completePayment('success');
                }
            });
        }
    }, [createPaymentRequestInterface, setPrInterface]);
    
    return (
        <div style={{ height: '100%' }}>
            { prInterface && <TecPaymentRequestButtons paymentRequestInterface={ prInterface } /> }
        </div>
    );
}
```

The above is a minimally viable solution to get you started. There are three more code examples - a [minimally viable example](#example-implementation) using both Apple Pay and Google Pay, as well as two more complex examples to leverage more of the features available, for each respective platform ([Apple Pay Complex Example here](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Apple-Pay-Example-Implementation), [Google Pay Complex Example here](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md#Google-Pay-Example-Implementation)). Read on for further features and details availble to the Payment Request Utility.  
<br />

# Payment Request Object
The Payment Request Object is the only parameter for the [```createPaymentRequestInterface```](#Create-Payment-Request-Interface) function. This object describes the form(s) that the end-user will interact with.  Each platform will have a different payment request object with different properties. If your application has opted-in for more than one platform (i.e. both Google and Apple Pay) - be sure to seperate the payment request objects by platform:  

```javascript
const applePaymentRequestObject = {
    storeDisplayName: "TEConnect Example Store",
    currencyCode: "USD",
    countryCode: "US",
    supportedNetworks: ['visa', 'masterCard', 'amex', 'discover', 'jcb'],
    merchantCapabilities: ['supports3DS'],
    total: {
        label: "Test Transaction",
        amount: "1.00",
        type: "final"
    }
};

const googlePaymentRequestObject = {
    allowedCardNetworks: ["AMEX", "DISCOVER", "JCB", "MASTERCARD", "MIR", "VISA"],
	merchantName: "TEConnect Example Merchant",
	gatewayId: "_gateway_id_provided_by_Magensa_",
	transactionInfo: {
		totalPriceStatus: 'FINAL',
		totalPrice: '1.23',
		currencyCode: 'USD',
		countryCode: 'US'
    }
};

const teConnectPaymentRequestObject = {
    applePay: applePaymentRequestObject,
    googlePay: googlePaymentRequestObject
};
```

### Apple Pay Payment Request Object
[More information and helpful links about Apple's Payment Request Object can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Apple-Pay-Payment-Request-Object)

### Google Pay Payment Request Object
[More information and helpful links about Google's Payment Request Object can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md#Google-Pay-Payment-Request-Object)

# Token Exchange Connect Payment Request Interface
The Token Exchange Connect Payment Request Interface (referred to as the ```tecPrInterface```) is the interface needed to interact, and respond to user's interactions on the Payment Request Form.

## Create Payment Request Interface
The ```useTecPaymentsRequest``` hook returns a function. This document refers to that function as the ```createPaymentRequestInterface``` - but this function can be named as desired.

## Interface Methods
### ```canMakePayments```
This function returns a Promise, that resolves to a [```CanMakePaymentsResult```](#CanMakePaymentsResult)
```javascript
import React from 'react';
import { useTecPaymentsRequest } from '@magensa/te-connect-react';

import { teConnectPaymentRequestObject } from '../consts';

const ExampleApp = () => {
    const createTecPrInterfaceMethod = useTecPaymentsRequest();

    useEffect(() => {
        if (typeof(createTecPrInterfaceMethod) === 'function') {
            const tecPrInterface = createTecPrInterfaceMethod(teConnectPaymentRequestObject);

            tecPrInterface.canMakePayments().then( availablePrMethods => {
                if (availablePrMethods) {
                    //User has ability to pay using a Payment Request platform
                }
            });
        }
    }, [createTecPrInterfaceMethod]);
}
```
#### ```CanMakePaymentsResult```
```typescript
type CanMakePaymentsResult = {
    applePay?: boolean,
    googlePay?: boolean
} | null;
```

### ```updatePaymentRequest```
This function is only useful _before_ the user has hit the Apple Pay button.  If you have [created a payment request interface](#Create-Payment-Request-Interface), using a [payment request object](#Payment-Request-Object) - but wish to update that object before the user hits the button and renders the payment request form - you may call this function to do so.  Be aware that this function is a __full update__, so the object will completely replace the previous payment request object supplied.
- It's not necessary to update your object inside of [payment request listeners](#Payment-Request-Event-Handlers). Any response in your completion functions will update the form dynamically.


### ```listenFor```
This function accepts an event name (```string```) to listen for, as well as a listener function. These events are specific to payment request interface instance created. Please [see the section below](#Payment-Request-Event-Handlers) for more details.


# Payment Request Event Handlers
Listening to the ```confirm-token``` event is required to complete the workflow for all payment platforms.

Apple Pay uses an event handler driven workflow. When users interact with the payment request form - there are several events that are fired.   
[More details about Apple Pay listeners here](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Apple-Pay-Listeners)

### ```confirm-token``` Event
This is the only event listener that is required to complete the both workflows. This is the only event that can optionally contain an ```error``` property (in the case the payment was submitted, but was unsuccessful). Be sure to check for that property first, if it exists.  
When listening to the ```confirm-token``` event - there will be up to two special properties to the event:
- ```tokenDetails```
    - ```type``` specifies the payment request platform in which the token was created.
        - ```applePay``` or ```googlePay```.
    - ```token``` will be the object needed to process transactions, using the payment token created during the current session.
        - Pass the ```token``` to the appropriate MPPG operation, unaltered, for processing.
    - ```error``` is an optional property in the case the token creation was unsuccessful.
- ```completePayment```
    - This function will only exist for Apple Pay tokens. It is used to close the Apple Pay form with a status message. 
        - [More information about ```completePayment``` can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Apple-Pay-Listeners)
        - This property will not exist for Google Pay tokens. This is one of the many ways to differentiate between which platform the user is using to complete the transaction.
            
```confirm-token``` example: 
```javascript
import React, { useState } from 'react';
import { useTecPaymentsRequest } from '@magensa/te-connect-react';

import { teConnectPaymentRequestObject } from '../consts';

const ExampleApp = () => {
    const createTecPrInterfaceMethod = useTecPaymentsRequest();
    const [ isAppleUser, setIsAppleUser ] = useState(false);

    useEffect(() => {
        if (typeof(createTecPrInterfaceMethod) === 'function') {
            const tecPrInterface = createTecPrInterfaceMethod(teConnectPaymentRequestObject);

            tecPrInterface.canMakePayments().then( availablePrMethods => {
                if (availablePrMethods && availablePrMethods.applePay === true) {
                    setIsAppleUser(true);
                }
            });

            tecPrInterface.listenFor('confirm-token', tokenResponse => {
                const { tokenDetails, completePayment, error } = tokenResponse;

                if (error) {
                    //You can complete with string value, as below.
                    //completePayment("failure");

                    //Optionally - you can provide more concise error messages
                    if (isAppleUser === true) {
                        completePayment({
                            status: ApplePaySession.STATUS_FAILURE,
                            errors: [
                                new ApplePayError("shippingContactInvalid", "postalCode", "ZIP Code is invalid"),
                                new ApplePayError("addressUnserviceable", "addressLines", "Cannot deliver to P.O. Box")
                            ]
                        })
                    }
                }
                else {
                    if (typeof( completePayment ) === 'function') {
                        //tokenDetails.type === 'applePay'
                        completePayment('success');
                        //pass `tokenDetails.token` to MPPG to process payment
                    }
                    else {
                        //tokenDetails.type === 'googlePay'
                        //pass `tokenDetails.token` to MPPG to process payment
                    }
                }
            });
        }
    }, [createTecPrInterfaceMethod, isAppleUser]);
}
```

# Payment Request Button Options
The ```TecPaymentRequestButtons``` component accepts an optional prop named ```tecPrOptions```.  Here you can define your custom options for the payment request buttons. You may define custom values for the Apple Pay Button, using the property ```applePayOptions```, and Google Pay Button with ```googlePayOptions``` (You can see the [default values](#Default-Payment-Request-Button-Options-Values), for an example on how to structure the object, [below]).

## Apple Pay Button Options
[More details about Apple Pay Button Options can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Apple-Pay-Button-Options)

## Google Pay Button Options
[More details about Google Pay Button Options can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md#Google-Pay-Button-Options)

#### Default Payment Request Button Options Values:  
  
```javascript
const defaultButtonOptions = {
    applePayOptions: {
        buttonLanguage: 'en',
        buttonType: 'plain',
        buttonStyle: 'black'
    },
    googlePayOptions: {
        preClick: undefined, //Only calls if is of type 'function'
        buttonColor: undefined, //Currently defaults to 'black' but default is not static, and is determined by Google
        buttonType: undefined, 
        buttonLocale: undefined, //If not supplied - defaults to browser or OS language settings
        buttonSizeMode: 'static'
    }
};
```


# Example Implementation
Below you will find a minimally viable implementation.  
  
```javascript
import React, { useState } from 'react';
import ReactDOM from 'react-dom';
import { TEConnect, TecPaymentRequestButtons, useTecPaymentsRequest } from '@magensa/te-connect-react';
import { createTEConnect } from '@magensa/te-connect';
import { exampleApplePaymentRequestObject, exampleGooglePaymentRequestObject } from '../consts';

const paymentRequestObject = {
    applePay: exampleApplePaymentRequestObject,
    googlePay: exampleGooglePaymentRequestObject
}

const teConnectInstance = createTEConnect("__publicKeyGoesHere__", { 
    tecPaymentRequest: { 
        appleMerchantId: "__tecAppleMerchantId__",
        googleMerchantId: "__googleMerchantId__"
    }
});

const ExampleApp = () => {
    const [ prInterface, setPrInterface ] = useState(null);
    const createTecPrInterfaceMethod = useTecPaymentsRequest();

    useEffect(() => {
        //Check to ensure creation function is available
        if (typeof(createTecPrInterfaceMethod) === 'function') {
            const tecPrInterface = createTecPrInterfaceMethod(examplePaymentRequestObject);

            tecPrInterface.canMakePayments().then( availablePrMethods => {
                if (availablePrMethods)
                    setPrInterface(tecPrInterface);
            });

            tecPrInterface.listenFor('confirm-token', (tokenResp) => {
                const { tokenDetails, completePayment, error } = tokenResp;

                if (error) {
                    //Unsuccessful - check message.
                    tokenResp.completePayment('failure');
                }
                else {
                    //Successful
                    //Pass `tokenDetails.token` directly to MPPG for processing.
                    //if Apple Pay - close user's payment request form with payment success notification.
                    if (typeof(completePayment) === 'function')
                        completePayment('success');
                }
            });
        }
    }, [createTecPrInterfaceMethod, setPrInterface]);
    
    return (
        <div style={{ height: '100%' }}>
            { prInterface && <TecPaymentRequestButtons paymentRequestInterface={ prInterface } /> }
        </div>
    );
}

const App = () => (
    <TEConnect teConnect={ teConnectInstance }>
        <ExampleApp />
    </TEConnect>
);

ReactDOM.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>,
    document.getElementById('root')
);
```

# Payment Request Error Handling
All Apple Pay callbacks accept an ```errors[]``` array. [More information here](https://github.com/Magensa/te-connect-react/blob/master/TecApplePayREADME.md#Apple-Pay-Error-Handling)  
  
Google Pay Errors are handled within the optional callbacks. [More information on that here](https://github.com/Magensa/te-connect-react/blob/master/TecGooglePayREADME.md#Google-Pay-Error-Handling)
