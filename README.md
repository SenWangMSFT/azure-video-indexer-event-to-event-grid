# Using Azure Logic Apps to Poll Video Indexer and Publish to Event Grid

This guide provides step-by-step instructions on setting up an Azure Logic App to periodically poll the Azure AI Video Indexer and publish events to Azure Event Grid when video indexing is complete.

## Prerequisites

- An Azure subscription.
- Azure AI Video Indexer account.
- Azure Event Grid Topic.
- Access keys for the above services.

## Steps

### 1. Create an Azure Logic App

1. Go to the Azure portal.
2. Search for and select "Logic Apps".
3. Click "Add" to create a new Logic App.
4. Fill in the necessary details and create the Logic App.

### 2. Design the Logic App

1. Once the Logic App is created, go to the Logic App Designer.
2. Choose the "Blank Logic App" template to start from scratch.

### 3. Add Recurrence Trigger

1. In the Logic App Designer, add the "Recurrence" trigger.
2. Set the frequency and interval (e.g., every 5 minutes) to check for new video indexing results.

### 4. Add an HTTP Action to Call Video Indexer API

1. Click on "New Step" and search for "HTTP".
2. Choose the "HTTP" action.
3. Configure the HTTP action to call the Azure AI Video Indexer API to get the status of video indexing jobs.
   - Method: GET
   - URI: `https://api.videoindexer.ai/{location}/Accounts/{account-id}/Videos/{video-id}/Index?accessToken={access-token}`

### 5. Parse the Video Indexer Response

1. Add a "Parse JSON" action to parse the response from the Video Indexer API.
2. Configure the action with the schema based on the Video Indexer API response.

### 6. Add a Condition to Check the Status

1. Add a "Condition" action to check if the video indexing is complete.
2. Configure the condition to check if the status in the parsed JSON is "Processed".

### 7. Add an HTTP Action to Publish to Event Grid

1. In the "If true" branch of the condition, add another HTTP action.
2. Configure the HTTP action to send the event to the Event Grid topic.
   - Method: POST
   - URI: Your Event Grid Topic Endpoint
   - Headers: Add "aeg-sas-key" with your Event Grid Topic Access Key
   - Body: Construct the JSON body for the Event Grid event.

#### Example JSON Body for Event Grid Event

```json
[
  {
    "id": "{event-id}",
    "eventType": "VideoIndexer.Processed",
    "subject": "/videoindexer/videos/{video-id}",
    "eventTime": "{event-time}",
    "data": {
      "videoId": "{video-id}",
      "indexingStatus": "Processed",
      "details": "{details}"
    },
    "dataVersion": "1.0"
  }
]
```

### 8. Finalizing the Logic App

1. **Save and Test the Logic App:**
   - Save the Logic App.
   - Upload a video to Azure AI Video Indexer and wait for the indexing to complete.
   - The Logic App will periodically poll the Video Indexer API, and upon detecting a completed indexing job, it will publish an event to Event Grid.

2. **Monitor the Logic App:**
   - Go to the Logic App's "Runs history" to monitor and troubleshoot any issues with the Logic App execution.

## Documentation Links

- [Microsoft Azure API Management - Developer Portal](https://videoindexer.ai)
- [Azure Event Grid event schema - Azure Event Grid](https://learn.microsoft.com/en-us/azure/event-grid/event-schema)
- [Create workflows that call external endpoints or other workflows - Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-create-and-call-azure)
- [Perform operations on data - Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-perform-data-operations)

## Conclusion

By following these steps, you can set up a Logic App to automatically handle video indexing status checks and publish relevant events to Azure Event Grid, enabling seamless integration with other Azure services.