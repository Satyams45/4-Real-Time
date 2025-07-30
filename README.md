
# 🚕 CST8917 Lab 4: Real-Time Trip Event Analysis

This lab guides you through implementing a real-time analytics pipeline using Azure services to process taxi trip events and send alerts via Microsoft Teams.

---

## 📌 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Step 1: Create Azure Resources](#step-1-create-azure-resources)
4. [Step 2: Build Azure Function to Process Event Hub Data](#step-2-build-azure-function-to-process-event-hub-data)
5. [Step 3: Configure Logic App to Post to Microsoft Teams](#step-3-configure-logic-app-to-post-to-microsoft-teams)
6. [Step 4: Test with Static Data](#step-4-test-with-static-data)
7. [Step 5: Simulate Event Data](#step-5-simulate-event-data)
8. [Output Example](#output-example)
9. [Diagram](#diagram)

---

## 🧭 Overview

The goal is to set up a pipeline that:
- Ingests trip events through Azure Event Hub
- Processes the data with an Azure Function
- Routes alerts through a Logic App
- Posts Adaptive Cards to a Microsoft Teams channel

This simulates a real-world use case for fleet or delivery service operations.

---

## 🏗️ Architecture Diagram

![Screenshot 2025-07-29 231906](https://github.com/user-attachments/assets/b467d8b5-6fd8-4a0b-a346-0e59230aa8e4)

---

## 🛠️ Step 1: Create Azure Resources

1. **Event Hub Namespace & Event Hub**  
   - Create a namespace: `trip-event-ns`  
   - Inside it, create an Event Hub named `trips`

2. **Azure Function App**
   - Use Python 3.10 runtime
   - Create a function `TripProcessor` triggered by Event Hub

3. **Azure Logic App**
   - Triggered by an HTTP request from the Azure Function
   - Posts formatted message to a Microsoft Teams channel using a webhook

---

## 💡 Step 2: Build Azure Function to Process Event Hub Data

Example function code:

```python
import logging
import azure.functions as func
import json
import requests
import os

def main(event: func.EventHubEvent):
    trip_data = json.loads(event.get_body().decode('utf-8'))
    logging.info(f"Received Trip Data: {trip_data}")

    teams_webhook_url = os.environ["TEAMS_WEBHOOK_URL"]

    card = {
        "type": "message",
        "attachments": [{
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                "type": "AdaptiveCard",
                "version": "1.3",
                "body": [
                    {"type": "TextBlock", "text": f"🚕 Trip Event: {trip_data['status']}", "weight": "Bolder", "size": "Medium"},
                    {"type": "FactSet", "facts": [
                        {"title": "Trip ID:", "value": trip_data["trip_id"]},
                        {"title": "Vehicle:", "value": trip_data["vehicle_id"]},
                        {"title": "Timestamp:", "value": trip_data["timestamp"]}
                    ]}
                ]
            }
        }]
    }

    headers = {"Content-Type": "application/json"}
    response = requests.post(teams_webhook_url, json=card, headers=headers)
    logging.info(f"Teams Response: {response.status_code}")
```

---

## 🔁 Step 3: Configure Logic App to Post to Microsoft Teams

1. Create a new Logic App (Consumption)
2. Add a trigger: **When an HTTP request is received**
3. Add an action: **Post message (V3) to Teams channel**
4. Use the webhook URL or adaptive card schema as needed

---

## 🧪 Step 4: Test with Static Data

Before simulating events, test manually:

```bash
curl -X POST "<logic-app-endpoint>"   -H "Content-Type: application/json"   -d '{"trip_id":"123", "vehicle_id":"TX-09", "status":"trip_started", "timestamp":"2025-07-30T20:00:00Z"}'
```

---

## 📡 Step 5: Simulate Event Data

### Requirements

- Python 3.8+
- Install Azure Event Hub SDK:

```bash
pip install azure-eventhub
```

### Python Script: `send_event.py`

```python
from azure.eventhub import EventHubProducerClient, EventData
import json

CONNECTION_STR = "Endpoint=sb://<your-namespace>.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<your-key>"
EVENTHUB_NAME = "trips"

producer = EventHubProducerClient.from_connection_string(
    conn_str=CONNECTION_STR,
    eventhub_name=EVENTHUB_NAME
)

event_data = {
    "trip_id": "1001",
    "vehicle_id": "TX-9876",
    "status": "trip_started",
    "timestamp": "2025-07-30T19:30:00Z"
}

event_batch = producer.create_batch()
event_batch.add(EventData(json.dumps(event_data)))
producer.send_batch(event_batch)
print("✅ Event sent successfully.")
```

> 🔐 **Important**: Use the connection string from "Shared Access Policies > RootManageSharedAccessKey"

---

## ✅ Output Example

A message will be posted in Teams like:

> **🚕 Trip Event: trip_started**  
> **Trip ID**: 1001  
> **Vehicle**: TX-9876  
> **Timestamp**: 2025-07-30T19:30:00Z

---

## 🎥 Video Link

Watch the project demo here: [https://your-video-link.com](https://youtu.be/NDcfRxa7LfQ)


## 🧹 Cleanup

- Delete the Event Hub, Function App, and Logic App from Azure Portal if no longer needed.

---


