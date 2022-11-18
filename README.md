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
This document will cover the card manual entry component that TEConnect offers.  
TEConnect also offers a [Payment Request](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) component as well, with both Apple Pay and Google Pay supported. [Payment Request Documentation can be found here](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md)  
Below we will begin with a step-by-step integration of the card manual entry component. If you would prefer to let the code speak, there is an [example implementation](#Example-Implementation).

## Magensa™
TEConnect Manual Entry, and [TEConnect Payment Request](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) components both require a valid [Magensa™](https://magensa.net/) account. If you need assistance creating or configuring an account, please reach out to the [Magensa Support Team](https://magensa.net/support.html).

# Step-By-Step

1. The first step is to create a TEConnect instance, and feed that instance to the wrapper around your application.   

    ```javascript
    import React from 'react';
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
  
2. Next, once you have your form designed - drop the ```CardEntry``` component in the place of your choosing.
    ```javascript
    import React from 'react';
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

 3. Finally, to submit the inputted card values, utilize the ```createPayment``` and ```getCurrentElements``` functions and attach to your click handler.  
 ```javascript
    import React from 'react';
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

- Note that in this example, we chose to provide customStyles to the ```CardEntry``` component.  
<br />

# TEConnect Options
The second parameter of the ```createTEConnect``` method is an options object. This object is optional.
| Property Name  | Input Type | Notes |
|:--:|:--:|:--:|
| billingZip | ```boolean``` | See [billingZip options below](#Options-for-billingZip) |
| tecPaymentRequest | ```TecPaymentRequestOptions``` | See the [Payment Request README for more info](https://github.com/Magensa/te-connect-react/blob/master/TecPaymentRequestREADME.md) |


```typescript
type TecPaymentRequestOptions = {
    appleMerchantId?: string,
    googleMerchantId?: string
}

type CreateTEConnectOptions = {
    hideZip?: boolean,
    tecPaymentRequest?: TecPaymentRequestOptions
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

# createPayment Return Objects
These are the possible objects that will be returned *successfully* from the ```createPayment``` function. Thrown errors will be thrown as any other async method.  
  
  1. ### Success:
```typescript
{
    magTranID: String,
    timestamp: String,
    customerTranRef: String,
    token: String,
    code: String,
    message: String,
    status: Number,
    cardMetaData: null | {
        maskedPAN: String,
        expirationDate: String,
        billingZip: null | String
    }
}
```  
  
  2. ### Bad Request
```typescript
{
    magTranID: String,
    timestamp: String,
    customerTranRef: String,
    token: null,
    code: String,
    message: String,
    error: String,
    cardMetaData: null
}
```

3. ### Error (Failed Validation, Timeout, Mixed Protocol, etc)
```typescript
{ error: String }
```

# Styles API
The styles object injected is composed of two main properties:
- ```base```  
    - General styles applied to the container.
- ```boxes```
    - Styles applied to the input elements.

Below we have the complete API with examples of default values for each.

### Base
| Property Name | Parent Property | Input Type | Acceptable Values | Default Value | Notes |
|:--:|:--:|:--:|:--:|:--:|:---:|
| backgroundColor | base | ```string``` | jss color (rgb, #, or color name) | ```"#fff"``` | container background color |
| margin | wrapper | ```string``` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container margin |
| padding | wrapper | ```string``` or ```number``` | jss spacing units (rem, em, px, etc) | ```'1rem'``` | container padding |
| direction | wrapper | ```string``` | ```'row', 'row-reverse', 'column', 'column-reverse'``` | ```'row'``` | ```'flex-direction'``` style property |
| flexWrap | wrapper | ```string``` | ```'wrap', 'wrap', 'wrap-reverse'``` | ```'wrap'``` | ```'flex-wrap'``` style property |
| inputType | variants | ```string``` | ```"outlined", "filled", "standard"``` | ```"outlined"``` | template design for input boxes |
| inputMargin | variants | ```string``` | ```"dense", "none", "normal"``` | ```"normal"``` | template padding & margins for input boxes |  
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


# Example Implementation

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
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
        <React.Fragment>
            <CardEntry stylesConfig={ customStyles } />
            
            <button onClick={ clickHandler }>Create Token</button>
        </React.Fragment>
    )
}

const teConnectInstance = createTEConnect("__publicKeyGoesHere__");

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


## Minimum Requirements
There are three dependencies to use this product. If you built your project using [create-react-app](https://github.com/facebook/create-react-app), then you have already met the requirements (may require a 'uuid' upgrade). If you are building your own React app, then please make sure to include the following in your project:  

- [React](https://github.com/facebook/react) ^16.8
- [react-dom](https://github.com/facebook/react/tree/master/packages/react-dom) ^16.8
- [uuid](https://github.com/uuidjs/uuid) ^8.3.0
