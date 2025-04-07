# TEConnect React Component
[![npm version](https://img.shields.io/npm/v/@magensa/te-connect-react.svg?style=for-the-badge)](https://www.npmjs.com/package/@magensa/te-connect-react "@magensa/te-connect-react npm.js")  

React component for use with Token Exchange Connect utility.

# Getting Started
```
npm install @magensa/te-connect @magensa/te-connect-react
```
or
```
yarn add @magensa/te-connnect @magensa/te-connect-react
```

# Manual Card Entry
This document will cover the card manual entry component that TEConnect offers. 3DS Manual Entry is also available, if a valid 3DS API Key is supplied to [the `options` parameter](#TEConnect-Options).  
TEConnect also offers a [Payment Request](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) component, with both Apple Pay and Google Pay supported. [Payment Request Documentation can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md)  
Below we will begin with a step-by-step integration of the card manual entry component. If you would prefer to let the code speak, there are [example implementations](#Example-Implementation) for both Manual Entry and 3DS Manual Entry.

## Magensa™
TEConnect Manual Entry, 3DS Manual Entry and [TEConnect Payment Request](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) components all require a valid [Magensa™](https://magensa.net/) account. If you need assistance creating or configuring an account, please reach out to the [Magensa Support Team](https://magensa.net/support.html).  

# Step-By-Step

1. The first step is to create a TEConnect instance, and feed that instance to the wrapper around your application.   

    ```javascript
    import { TEConnect } from '@magensa/te-connect-react';
    import { createTEConnect } from '@magensa/te-connect';
    import ExampleApp from './components/exampleApp';

    const teConnectInstance = createTEConnect("__publicKeyGoesHere__", /* { hideZip: true } */);

    const App = () => (
        <TEConnect teConnect={ teConnectInstance }>
            <ExampleApp />
        </TEConnect>
    );
    ```
    - It's recommended to place this instance at the entrypoint of your application, to avoid multiple re-renders - or creating a new instance accidentally.  
    - There is an [optional ```options``` object](#TEConnect-Options) that can be passed to ```createTEConnect``` as the second parameter. 
        - See [Billing ZIP Options](#Options-for-billingZip) for more info.
        - See [Payment Request](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) for more info.
        - See [TecThreeDs](#TecThreeDs-3DS) for more info.
  
2. Next, once you have your form designed - drop the ```CardEntry``` component in the place of your choosing.
    ```javascript
    import { CardEntry } from '@magensa/te-connect-react';

    const appStyles = {
        height: '100px'
    }

    export default () => (
        <form>
            <input type="text" name="customer-name" />
            <div styles={ appStyles }>
                <CardEntry />
            </div>
        </form>
    );

    ```

    - It's recommended to wrap the ```CardEntry``` with a ```<div>``` of your choosing - so you may apply wrapper styles directly to your own div for juxtaposition. It's important to note that the ```height``` of this component is set to ```100%``` so that it may respond to your application's wrapper styles. As for styles applied to the component itself - we have a [styles object](#Styles-API) that you may use to inject your own custom styles.  
    - ```stylesConfig``` prop is optional. If provided, it will override the defaults.

3. Finally, to submit the inputted card values, use the hook `useTeConnect` to provide the `createPayment` and `getCurrentElements` functions. Utilize the functions to your click handler, as demonstrated below.  
    ```javascript
        import { CardEntry, useTeConnect } from '@magensa/te-connect-react';

        const customStyles =  {
            base: {
                wrapper: {
                    margin: '2rem',
                    padding: '2rem'
                },
                variants: {
                    inputType: 'outlined',
                    inputMargin: 'dense'
                },
                backgroundColor: '#ff7961'
            },
            boxes: {
                textColor: "#90ee02",
                borderRadius: 4,
                errorColor: "#2196f3"
            }
        }

        export default () => {
            const { createPayment, getCurrentElements } = useTeConnect();

            const clickHandler = async(e) => {
                try {
                    const elements = getCurrentElements();
                    const teConnectResponse = await createPayment(elements /*,  billingZipCode */);
                    const { error } = teConnectResponse;

                    if (error)
                        console.log("Unsuccessful, message reads: ", error);
                    else
                        console.log('result:', teConnectResponse);
                }
                catch(err) {
                    console.log('[Catch]:', err);
                }
            }

            return (
                <>
                    <CardEntry stylesConfig={ customStyles } />
                    
                    <button onClick={ clickHandler }>Create Token</button>
                </>
            )
        }
    ```
- Make sure to ```await``` this call, since it is asyncronous, and additionally be sure to wrap the call in a ```try/catch```.
    - It's important to note that while there are some cases which will throw an exception (```catch```) - there are other cases in which an object will return successfully with an error message. Make sure you check for an ```error``` property on the returned object. If it's not present - the call is succesful. For additional information, please see the possible [return objects](#createPayment-Return-Objects)

- Note that in this example, we chose to provide `customStyles` to the ```CardEntry``` component.  
<br />

# TEConnect Options
The second parameter of the ```createTEConnect``` method is an options object. This object is optional.
| Property Name  | Input Type | Notes |
|:--:|:--:|:--:|
| billingZip | ```boolean``` | See [billingZip options below](#Options-for-billingZip) |
| tecPaymentRequest | ```TecPaymentRequestOptions``` | See the [Payment Request README for more info](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) |
| threeds | `TecThreeDsOptions` | See [TecThreeDs](#TecThreeDs-3DS) for more info |


```typescript
type TecPaymentRequestOptions = {
    appleMerchantId?: string,
    googleMerchantId?: string
}

type TecThreeDsOptions = {
    threedsApiKey: string,
    threedsEnvironment: "sandbox" | "production"
}

type CreateTEConnectOptions = {
    hideZip?: boolean,
    tecPaymentRequest?: TecPaymentRequestOptions,
    threeds?: TecThreeDsOptions
}
```

## Options for ```billingZip```
-----------------------  
Billing Zip Code (```billingZip```) is an optional field. There are two different ways to supply the ```billingZip```, and one way to opt out.  
### _Opt In_:
The below instance will display the "ZIP Code" field along the other inputs. When choosing this option, the ZIP Code input box will be a required field for the customer.
```javascript
    const teConnectInstance = createTEConnect("__publicKeyGoesHere__");
```  

### _Supply Optional Billing Zip_:
The below will hide the "ZIP Code" input box. Once the field is hidden - the value becomes optional. If you would still like to supply a billing zip (I.E. You have a ZIP Code input on your own form, and would like the billingZip to be included in the payment token), here is an example of how to achieve that.
```javascript
    //First, create the instance with the ZIP field hidden
    const teConnectInstance = createTEConnect("__publicKeyGoesHere__", { hideZip: true });

    //Then supply the value when making the call to create payment token.
    const clickHandler = async(e) => {
        const billingZipCode = "90018";
        //Note that 'billingZip' is a string.

        try {
            const elements = getCurrentElements();
            const teConnectResponse = await createPayment(elements, billingZipCode);
            const { error } = teConnectResponse;

            if (error)
                console.log("Unsuccessful, message reads: ", error);
            else
                console.log('result:', teConnectResponse);
        }
        catch(err) {
            console.log('[Catch]:', err);
        }
    }
```  

### _Opt Out_:
The below will hide the input field, and when a value is not supplied - the payment token will be created without a billing zip code.  
```javascript
    //First, create the instance with the ZIP field hidden
    const teConnectInstance = createTEConnect("__publicKeyGoesHere__", { hideZip: true });

    //When the ZIP field is hidden, the billingZip parameter is optional.
    const clickHandler = async(e) => {
        try {
            const elements = getCurrentElements();
            const teConnectResponse = await createPayment(elements);
            const { error } = teConnectResponse;

            if (error)
                console.log("Unsuccessful, message reads: ", error);
            else
                console.log('result:', teConnectResponse);
        }
        catch(err) {
            console.log('[Catch]:', err);
        }
    }
```  
<br />

# TecThreeDs (3DS)
TEConnect offers a 3DS Manual Entry Component: `<ThreeDsCardEntry />`. To opt-in:
1. Add a `threeds` parameter to the [the ```options``` object](#TEConnect-Options) for the `createTEConnect` method.
2. Add a 3DS API key (`threedsApiKey`) to the `threeds` object, and (optionally) a `threedsEnvironment` string as well (defaults to `sandbox` for testing, when not specified). 
    - Only flip your project to `production` when you have tested with `sandbox`, and are ready to deploy to production.
3. Use the `ThreeDsCardEntry` component, in the same manner you would use the `CardEntry` component.
    - Only one Manual Entry component may be mounted to the DOM at a time.  If your use-case uses both `CardEntry` and `ThreeDsCardEntry` - ensure there is a condition in place that will only mount one or the other.

[3DS `sandbox` Test Cards can be found here](https://docs.3dsintegrator.com/docs/test-cards#emv-3ds-test-cards).

### _Opt In_
```javascript
    const teInstance = createTEConnect("__publicKeyGoesHere__", { 
        threeds: {
            threedsApiKey: "__3dsApiKeyGoesHere__",
            threedsEnvironment: "sandbox"
        }
    });
```  

3DS workflow is used in a similar manner as manual entry. There are a few additional requirements: 
- A required [`threeDsConfig`](#ThreeDsConfigObject) object must be fed to the `<ThreeDsCardEntry />` component.
    - For 3DS use the `useTecThreeds` hook instead of the manual entry `useTeConnect` hook.
- Use `ThreeDsCardEntry` instead of the `CardEntry` component.


```javascript
import { ThreeDsCardEntry, useTecThreeds } from '@magensa/te-connect-react';

const exampleThreedsConfig = {
  amount: 110,
  challengeNodeId: "example-challenge-node"
};

const App = () => {
  const { createPayment, getCurrentElements } = useTecThreeds();

    const clickHandler = async(e) => {
        try {
            const elements = getCurrentElements();
            console.log(elements);
            const teConnectResponse = await createPayment(elements /* billingZipCode */);
            const { error } = teConnectResponse;

            if (error)
                console.log("Unsuccessful, message reads: ", error);
            else
                console.log('result:', teConnectResponse);
        }
        catch(err) {
            console.log('[Catch]:', err);
        }
    }

    return (
    <>
        <ThreeDsCardEntry threeDsConfig={ exampleThreedsConfig } />
                
        <button onClick={ clickHandler }>Create Token</button>
    </>
  )
}

```
For a minimally viable 3DS solution, those are the only additional requirements needed complete the manual entry workflow. However, there is a `threeDsInterface` returned from the `useTecThreeds` hook, which can optionally be used for more methods.  

See the [TecThreeDs Example](#TecThreeDsExample) for an example implementation.

### `threeDsInterface`
| Method Name  | Parameters | Notes |
|:--:|:--:|:--:|
| listenFor | eventType (as `string`),  listener (as `function`) | Currently, `"threeds-status"` is the only `eventType` available to listen for [3DS status updates](#ThreeDs-Status-Listeners) |
| updateThreedsConfig | `ThreeDsConfigObject` | Full update (complete object). Updates are only available prior to 3DS auth. Once cardholder has entered card info, object is not able to be updated |

See the [TecThreeDs Example](#TecThreeDsExample) for example uses for these methods.

## `ThreeDsConfigObject`
TEConnect forwards the `threeDsConfig` object to 3DS API call 'authenticate browser'. This method begins the 3DS process; TEConnect handles the interactions on your application's behalf while the cardholder is entering the card info. TEConnect only requires two properties to get started: 
- `challengeNodeId: string` - the id of a node that must be mounted on the DOM. This node is used to mount a "challenge" iframe. 
    - The challenge iframe is defined by the issuer of the card being authenticated (in the event that a "challenge" is required to complete the 3DS authentication). This challenge varies based upon the issuer of the card. [See ThreeDsChallenge for more info](#ThreeDs-Challenge)
- `amount: number` - the final amount for the transaction. Required for 3DS Auth. 

Please note: There are four properties, documented in the [3DS API documentation](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser), that TEConnect handles and will be ignored if supplied in the ThreeDsConfigObject: `browser` object, and card info (`pan`, `year`, and `month`).

There are many optionally properties available, in addition to the required `challengeNodeId` and `amount` properties. 3DS has extensive documentation on the properties available in this call, which [can be found here](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser).  With the exception of the `browser` object, `pan`, `year`, and `month` - all other options supplied to the `<ThreeDsCardEntry />` component will be forwarded to the 3DS API call.


## ThreeDs Status Listeners
### `threeds-status`
Subscribe to this event to listen for status updates with the 3DS workflow.  
See the [TecThreeDs Example](#TecThreeDsExample) for an example how to subscribe.

```typescript
type ThreeDsMethods = "GENERATE_JWT" | "AUTHENTICATE_BROWSER" | "FINGERPRINT_DEVICE" | "CHALLENGE";
type ThreeDsStatus = "REQUESTED" | "SUCCESS" | "FAIL" | "AWAIT_RESULTS";

type ThreeDsStatusEvent = {
    threedsMethod: ThreeDsMethods,
    status: ThreeDsStatus,
    message: string
}
```

## ThreeDs Challenge
There are many possible 3DS responses ([3DS documents responses here](https://docs.3dsintegrator.com/reference/subscribetoupdates)).  When a status of type `"C"` is returned - this means that the card issuer has requested the cardholder to complete a challenge, in order to proceed. In this case - TEConnect will build the challenge iframe, and mount it on the node with the id provided (via `challengeNodeId`).  

The particular challenge rendered varies greatly depending on card issuer. The cardholder must complete the challenge in order to proceed with 3DS Authentication.

There are a few options available in the [ThreeDsConfigObject](https://docs.3dsintegrator.com/reference/post_v2-2-authenticate-browser), in regards to a challenge.  Such as `challengeIndicator` to specify if a challenge is desired.  Other options include `challengeWindowSize` and `transactionForcedTimeout` (if an early timeout is desired).  

TEConnect will attempt to collect the challenge results 10 seconds after a challenge is rendered. If the results are not yet available, it will poll until results are available or timeout. It will timeout after 60 seconds.

Since it's unknown what the challenge will look like, as well as the operations needed to complete the challenge - it's recommended to mount the challenge on a node that is styled with `absolute` positioning.  If you wish to place the challenge in a modal - make use of the [`threeds-status`](#ThreeDs-Status-Listeners) and listen for the status: `{ threedsMethod: "CHALLENGE", status: "REQUESTED" }` to trigger the modal.  

## Additional ThreeDs Considerations
3DS can have many potential responses ([3DS documents responses here](https://docs.3dsintegrator.com/reference/subscribetoupdates)). While 3DS auth may or may not be successful, or approved - please note that a token is always returned with a successful response, and can be used to call MPPG's `ProcessToken` for processing.  It's up to the merchant whether or not to assume the risk in the transaction. You may want to be familiar with responses, as they relate to various card networks, to determine if a liability shift has occured.  For example: [Visa](https://docs.3dsintegrator.com/docs/visa), [MasterCard](https://docs.3dsintegrator.com/docs/mastercard#faud-liability-shift-conditions), etc.  

When `"sandbox"` environment is in use - the `threeDSRequestorURL` will use a placeholder value. When flipped to `"production"` - the origin of your web application will be used.  This is because `http:` and `localhost` domains fail validation for 3DS calls, but are useful for early development.  Be aware that your `"production"` web application must be deployed using a valid `https://` domain, for 3DS to be successful.

<br />  

# Styles API
This Styles API is available to customize the form rendered for manual entry.  You may also style the outer container itself, using CSS, by targeting the id: `"__te-connect-secure-window"`.

The styles object injected is composed of two main properties:
- ```base```  
    - General styles applied to the container.
- ```boxes```
    - Styles applied to the input elements.

Below we have the complete API with examples of default values for each.

### Base
| Property Name | Parent Property | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|:---:|
| backgroundColor | base | `string` | jss color (rgb, #, or color name) | ```"#fff"``` | container background color |
| margin | wrapper | `string` or `number` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container margin |
| padding | wrapper | `string` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container padding |
| direction | wrapper | ```string``` | ```'row', 'row-reverse', 'column', 'column-reverse'``` | ```'row'``` | ```'flex-direction'``` style property |
| flexWrap | wrapper | ```string``` | ```'wrap', 'wrap', 'wrap-reverse'``` | ```'wrap'``` | ```'flex-wrap'``` style property |
| inputType | variants | ```string``` | ```"outlined", "filled", "standard"``` | ```"outlined"``` | template design for input boxes |
| inputMargin | variants | ```string``` | ```"dense", "none", "normal"``` | ```"normal"``` | template padding & margins for input boxes |  
| inputSize | variants | ```string``` | ```"small", "medium"``` | ```"medium"``` | input component size |  
| autoMinHeight | variants | ```boolean``` | ```boolean``` | ```false``` | ```true``` will maintain a static margin on each input box that will not grow with validation errors | 
  
<br />

### Default Base Object:
```javascript
{
    base: {
        wrapper: {
            margin: '1rem',
            padding: '1rem',
            direction: 'row',
            flexWrap: 'wrap'
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'normal',
            inputSize: 'medium',
            autoMinHeight: false
        },
        backgroundColor: '#fff'
    }
}
```
<br />

### Boxes
| Property Name | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|
| labelColor | ```string``` | jss color (rgb, #, or color name) | ```"#3f51b5"``` | label text and input outline (or underline) color |
| textColor | ```string``` | jss color (rgb, #, or color name) | ```"rgba(0, 0, 0, 0.87)"``` | color of text for input value *Also applies :onHover color to outline/underline* |
| borderRadius | ```number``` | numerical unit for css ```border-radius``` property | 4 | border radius for input boxes |
| inputColor | ```string``` | jss color (rgb, #, or color name) | ```"#fff"``` | input box background color |  
| errorColor | ```string``` | jss color (rgb, #, or color name) | ```"#f44336"``` | Error text and box outline (or underline) color |  
  
<br />

### Default Boxes Object:
```javascript
{
    boxes: {
        labelColor: "#3f51b5",
        textColor: "rgba(0, 0, 0, 0.87)",
        borderRadius: 4,
        errorColor: "#f44336",
        inputColor: "#fff"
    }
}
```

# createPayment Return Objects
These are the possible objects that will be returned *successfully* from the ```createPayment``` function. Thrown errors will be thrown as any other async method.  
  
  1. ### Success:
```typescript
{
    magTranID: string,
    timestamp: string,
    customerTranRef: string,
    token: string,
    code: string,
    message: string,
    status: number,
    cardMetaData: null | {
        maskedPAN: string,
        expirationDate: string,
        billingZip: null | string
    },
    threedsResults?: ThreeDsResults
}
```  
  
  2. ### Bad Request
```typescript
{
    magTranID: string,
    timestamp: string,
    customerTranRef: string,
    token: null,
    code: string,
    message: string,
    error: string,
    cardMetaData: null
}
```

3. ### Error (Failed Validation, Timeout, Mixed Protocol, etc)
```typescript
{ error: string }
```

4. ### ThreeDsResults (property of Success response, if 3DS opt-in)  
    All responses documented in the [3DS Documentation](https://docs.3dsintegrator.com/reference/subscribetoupdates)  

```typescript
type ThreeDsResults = {
    scaRequired?: bool,
    creq?: string,
    status?: string,
    authenticationValue?: string,
    authenticationType?: string,
    eci?: string,
    acsUrl?: string,
    dsTransId?: string,
    acsTransId?: string,
    sdkTransId?: string,
    error?: string,
    errorDetail?: string,
    errorCode?: string,
    cardholderInfo?: string,
    transactionId?: string,
    correlationId?: string,
    code?: number
}
```


# Example Implementation

```javascript
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { TEConnect, CardEntry, useTeConnect  } from '@magensa/te-connect-react';
import { createTEConnect } from '@magensa/te-connect';

const customStyles =  {
    base: {
        wrapper: {
            margin: '2rem',
            padding: '2rem'
        },
        variants: {
            inputType: 'outlined',
            inputMargin: 'dense'
        },
       backgroundColor: '#ff7961'
    },
    boxes: {
        labelColor: "#9400D3",
        textColor: "#90ee02",
        borderRadius: 10,
        errorColor: "#2196f3",
        inputColor: '#ff7961'
    }
}

const ExampleApp = () => {
    const { createPayment, getCurrentElements } = useTeConnect();

    const clickHandler = async(e) => {
        try {
            const elements = getCurrentElements();
            const teConnectResponse = await createPayment(elements);
            console.log('result:', teConnectResponse);
        }
        catch(err) {
            console.error(err);
        }
    }
    
    return (
        <>
            <CardEntry stylesConfig={ customStyles } />
            
            <button onClick={ clickHandler }>Create Token</button>
        </>
    )
}

const teConnectInstance = createTEConnect("__publicKeyGoesHere__");

const App = () => (
    <TEConnect teConnect={ teConnectInstance }>
        <ExampleApp />
    </TEConnect>
);

createRoot(document.getElementById('root')).render(
    <StrictMode>
        <App />
    </StrictMode>,
    document.getElementById('root')
);
```

## TecThreeDs Example
This example uses the `threeDsInterface` methods. Note that this is optional, and not required to create a token with 3DS response.

 ```javascript
    import { useEffect } from 'react';
    import { ThreeDsCardEntry, useTecThreeds } from '@magensa/te-connect-react';

    const exampleThreedsConfig = {
        amount: 110,
        challengeNodeId: "example-challenge-node"
    };

    const threeDsStatusListener = msg => {
        console.log('[3DS Listener]:', msg);
    }

    const customStyles = {
        base: {
            variants: {
                inputType: 'outlined',
                inputMargin: 'dense'
            },
            backgroundColor: 'rgb(255, 255, 255);'
        },
        boxes: {
            labelColor: 'hsl(240 3.7% 15.9%)',
            borderRadius: 10,
            inputColor: 'hsl(240 3.7% 15.9%)',
            textColor: 'hsl(240 3.7% 15.9%)'
        }
    };

    export default () => {
        const { createPayment, getCurrentElements, threeDsInterface } = useTecThreeds();

        useEffect(() => {
            if (threeDsInterface)
                threeDsInterface.listenFor("threeds-status", threeDsStatusListener);
        }, [threeDsInterface]);

        const clickHandler = async(e) => {
            try {
                const elements = getCurrentElements();
                const teConnectResponse = await createPayment(elements, /* billingZipCode */);
                const { error } = teConnectResponse;

                if (error)
                    console.log("Unsuccessful, message reads: ", error);
                else
                    console.log('result:', teConnectResponse);
            }
            catch(err) {
                console.log('[Catch]:', err);
            }
        }

        const updateThreedsConfigObj = () => {
            if (threeDsInterface)
                threeDsInterface.updateThreedsConfig({
                    ...exampleThreedsConfig,
                    amount: 120
                });
        }

        return (
            <>
                <ThreeDsCardEntry threeDsConfig={ exampleThreedsConfig } stylesConfig={ customStyles } />
                
                <button onClick={ clickHandler }>Create Token</button>
                <button onClick={ updateThreedsConfigObj }>Update ThreeDs Config</button>
            </>
        )
    }
 ```


## Minimum Requirements
There are three dependencies to use this product. If you built your project using [create-react-app](https://github.com/facebook/create-react-app), then you have already met the requirements (may require a 'uuid' upgrade). If you are building your own React app, then please make sure to include the following in your project:  

- [React](https://github.com/facebook/react) ^18.1
- [react-dom](https://github.com/facebook/react/tree/master/packages/react-dom) ^18.1
- [uuid](https://github.com/uuidjs/uuid) ^8.3.0
