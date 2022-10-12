
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