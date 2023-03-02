# Azure Integration Services - Hands-on Labs

## Pre-requisites
1. Active Azure Subscription (get one [here](https://azure.microsoft.com/en-us/free/)).
2. \[Part 1\] A OneDrive account (M365 or personal)
3. \[Part 1\] An OpenAI account (either [Azure OpenAI](https://azure.microsoft.com/en-us/products/cognitive-services/openai-service) or directly from [OpenAI](https://platform.openai.com/signup))
4. \[Part 1, Optional\] [A SendGrid account](https://sendgrid.com/free/)
5. \[Part 2\] An [Azure API Management](https://learn.microsoft.com/en-us/azure/api-management/get-started-create-service-instance). This is for Part 2, but this 30-120 mins to provision, so best start now.


## Hands-on Lab

### Part 1: Azure Logic Apps - HTTP Trigger
> Idea: Create a new REST API using Azure Logic Apps that
> - Takes a text input
> - Create an OpenAI prompt text: combining a text content from OneDrive, and the request text body
> - Call OpenAI completion API, using text-davinci-002
> - \[Optional\] Send the result to an e-mail address
> - Return the result as an API Response

### Part 2: Azure API Management
> Idea: Use APIM as an abstraction layer (gateway) between the API consumer and the Azure Logic App.
> - Configure the API policies
> - Test the API
> - Explore the Developer Portal