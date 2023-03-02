# Azure Integration Services - Hands-on Labs

## Pre-requisites
1. Active Azure Subscription (get one [here](https://azure.microsoft.com/en-us/free/)).
2. \[Part 1\] A OneDrive account (M365 or personal)
3. \[Part 1\] An OpenAI account (either [Azure OpenAI](https://azure.microsoft.com/en-us/products/cognitive-services/openai-service) or directly from [OpenAI](https://platform.openai.com/signup))
4. \[Part 1, Optional\] [A SendGrid account](https://sendgrid.com/free/)
5. \[Part 2\] An [Azure API Management](https://learn.microsoft.com/en-us/azure/api-management/get-started-create-service-instance). This is for Part 2, but this 30-120 mins to provision, so best start now.

> **FOR TRAINERS:** You may pre-create items 1, 3, 4 and 5 above to make the lab faster.
> - Create lab user accounts, with access to an Azure Resource Group.
> - Pre-provision an Azure API management in this resource group.
> - Provide the OpenAI API URL and API key.
> - Provide the SendGrid API URL and API key.

## Hands-on Lab

### Challenge 1: Creating the Azure Logic App
> **IDEA:** Create a new REST API using Azure Logic Apps that...
> - Takes a text input
> - Create an OpenAI prompt text: combining a text content from OneDrive, and the request text body
> - Call OpenAI completion API, using text-davinci-002
> - \[Optional\] Send the result to an e-mail address
> - Return the result as an API Response

#### **1a: Create a Text file with content in OneDrive**
1. On your local machine, create a text file with the filename **/<your-name/>.txt**
2. In the file, enter a fun fact about yourself that the internet does not know. Avoid using special characters like the following `/ \ ' " ; < >` For example
    ```
    YOUR-NAME likes to drink coffee before sleeping.
    ```
3. Go to https://onedrive.live.com/ and login with the demo account provided (or your own preferred account)
4. Upload the text file that you created in #2.

#### **1b: Create a Logic App**
1. Sign into https://portal.azure.com
2. Create a new Azure Logic App resource
    - Resource name: /<your-name-abbreviation/>-logicapp
    - Region: West US
    - Resource Group: Your assigned resource group (or create a new one)
3. On the left, Click **Logic app designer**
4. Select **"When a HTTP request is received"**
5. Click on **"Use sample payload to generate schema"** and enter the following
    ```
    {"message":"Hello"}
    ```
6. It should look like this ![](screenshots/1-trigger.png)
7. Add a step **Initialize Variable**, which is under **Variables** ![](screenshots\1-init-variable.png)
8. Add a step and look for OneDrive (for personal accounts) or OneDrive for Business (for M365 accounts)
9. Select the action **Get file content** ![](screenshots/1c-onedrive.png)
10. Sign In with the OneDrive account you used in [1a](#1a-create-a-text-file-with-content-in-onedrive)
11. In *File, browse for your text file ![](screenshots/1-onedrive-file.png)
12. Add a step **Compose**, which is under **Data Operations**
13. Add as input the OneDrive **File content** and the HTTP Request **message**. It should look like this ![](screenshots/1-compose-prompt.png)
14. Add a step **HTTP** and enter the following:
    ```
    Method: POST

    // Change to the OpenAI completion URL if that's what you're using.
    URL: https://{your-resource-name}.openai.azure.com/openai/deployments/{deployment-id}/completions?api-version=2022-12-01

    Headers:
    1. api-key: {your-api-key}
    2. Content-Type: application/json

    Body:
    {
        "prompt":"{Outputs from #12}",
        "max_tokens": 1000
    }
    ```
15. Here's an example: ![](screenshots/1-http-openai.png)
16. Add a step **Parse JSON**, under **Data Operations**
17. In the content, select the **Body** from the **HTTP** action
18. In the Schema, copy-paste the following
    ```
    {
        "type": "object",
        "properties": {
            "id": {
                "type": "string"
            },
            "object": {
                "type": "string"
            },
            "created": {
                "type": "integer"
            },
            "model": {
                "type": "string"
            },
            "choices": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "text": {
                            "type": "string"
                        },
                        "index": {
                            "type": "integer"
                        },
                        "finish_reason": {
                            "type": "string"
                        },
                        "logprobs": {}
                    },
                    "required": [
                        "text",
                        "index",
                        "finish_reason",
                        "logprobs"
                    ]
                }
            },
            "usage": {
                "type": "object",
                "properties": {
                    "completion_tokens": {
                        "type": "integer"
                    },
                    "prompt_tokens": {
                        "type": "integer"
                    },
                    "total_tokens": {
                        "type": "integer"
                    }
                }
            }
        }
    }
    ```
19. Add a step **Append to string variable**, under **Variables**
20. For the name, select _Response_
20. For the value, select **text** from the **Parse JSON** action. This will automatically place this action in a For each loop. ![](screenshots/1-appendoutput.png)
21. Finally, add a step **Response**, under **Request** with the following values
    ```
    Status Code: 200

    Headers:
    1. Content-Type: application/json

    Body:
    {"response":"{response-variable}"}
    ```

**If your logic app looks like this, congratulations! You've completed Challenge 1.**
![logic-app-final](screenshots/1-final.png)

### Part 2: Azure API Management
> **IDEA:** Use APIM as an abstraction layer (gateway) between the API consumer and the Azure Logic App.
> - Configure the API policies
> - Test the API
> - Explore the Developer Portal