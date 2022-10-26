# joi-messages-extractor

This script creates joiErrorMessages.ts file which contain all possible error messages which could be generated by Joi and exports them like *joiErrorMessages*. *joiErrorMessages* object can be used for custom translations of Joi error messages.

Tested versions:
* 17.6.4 +
* 17.6.3 +
* 17.6.0 +
* 17.5.0 +
* 17.4.0 +
* 17.3.0 +
* 17.2.0 +
* 17.1.1 - Problem with TypeScript types. Error message: Could not find a declaration file for module 'joi'
## Usage

Script is using environment variables:

- JOI_MESSAGES_FILE_PATH - which describes the folder where joiErrorMessages.ts will be generated. **By default** the same folder with script
- JOI_CHECK_ONLY - Is a boolean. *false* - generate joiErrorMessages.ts, *true* - check translations and if there are missing some throw an error. **By default** is *false*

For running this script is used the following command:

``npx GoodRequest/joi-messages-extractor``

But this script is recommended to install with npm: 

``npm install GoodRequest/joi-messages-extractor``

And use like:

``npx joi-messages-extractor``

Usage example with env variables in command:

``cross-env JOI_MESSAGES_FILE_PATH=./locales JOI_CHECK_ONLY=false npx GoodRequest/joi-messages-extractor``

## Error messages file - joiErrorMessages.ts

joiErrorMessages.ts - is a default filename where all Joi error message keys with default translations are generated and exported. With common usage this file will never be overwritten, it will be appended with missing types only. File has the following structure:

```typescript
const joiErrorMessages: any = {
    en: {
        'any.custom': '{{#label}} failed custom validation because {{#error.message}}',
        'any.default': '{{#label}} threw an error when running default method',
        ...
    },
    sk: {
        'any.custom': '{{#label}} failed custom validation because {{#error.message}}',
        'any.default': '{{#label}} threw an error when running default method',
        ...
    },
    ...
}
export default joiErrorMessages
```
It contains keys *.en* and *.sk* which describe translation language. Script is trying to find them in *i18next.fallbackLng* from config module. Default values are *.en* and *.sk*.

Each key contains type of Joi error message, and it's translation. By default all messages are in English and must be translated manually.

## Usage of joiErrorMessages.ts in code

To use custom error messages is enough to pass *joiErrorMessages* object to Joi validation methode:

```typescript
import joiErrorMessages from 'joiErrorMessages.ts' // Importing our generated messages
import i18next from 'i18next' // Importing i18next

const result = schema.validate(validationValue, { 
    messages: joiErrorMessages, // Passing joiErrorMessages object to Joi
    errors: { 
        language: i18next.resolvedLanguage // Telling Joi which langage to use
    } 
})
```

As a result, in case of error we'll get a custom error message from Joi. Such patterns like *{{#label}}* or *{{#limit}} key{if(#limit == 1, "", "s")}* will be compiled by Joi itself.

## Usage script in pipelines

To avoid mistakes in project this script could be usefully to use in pipelines, for example *bitbucket-pipelines.yml*

```yaml
image: node:10.15.0
   
pipelines:
  default:
    - step:
        name: Build and test
        script:
          - npm install
          - npm test
          - npm joi:check
```

*joi:extract* in *package.json*:

```json
"scripts": {
    "joi:check": "cross-env JOI_MESSAGES_FILE_PATH=./locales JOI_CHECK_ONLY=true npx joi-messages-extractor",
    ...
},
...
```