# API Configuration
base_url: "https://semantic-brandea-banao-dc049ed0.koyeb.app"
credentials:
  username: "your_username"
  password: "your_password"

# Test Data
target_vs_name: "backend-vs-t1r_1000-1"
parallel_execution: true

import requests
import yaml
import threading
from requests.auth import HTTPBasicAuth

class AviAutomationFramework:
    def __init__(self, config_path='config.yaml'):
        with open(config_path, 'r') as file:
            self.config = yaml.safe_load(file) # Parse YAML dynamically [cite: 27]
        self.base_url = self.config['base_url']
        self.token = None

    def login(self):
        # Step 2: Login to get Bearer Token [cite: 63, 66]
        user = self.config['credentials']['username']
        pw = self.config['credentials']['password']
        response = requests.post(f"{self.base_url}/login", auth=HTTPBasicAuth(user, pw))
        if response.status_code == 200:
            self.token = response.json().get('token')
            print(f"[AUTH] Login successful. Token acquired.") [cite: 66]
        return self.token

    def get_headers(self):
        return {"Authorization": f"Bearer {self.token}", "Content-Type": "application/json"} [cite: 71, 72]

    # Mock components 
    def mock_ssh_connect(self, host):
        print(f"MOCK_SSH: Connecting to host {host}...") [cite: 32]

    def mock_rdp_validate(self):
        print("MOCK_RDP: Validating remote connection...") [cite: 33]

    # Stage 1: Pre-Fetcher [cite: 35]
    def pre_fetcher(self):
        print("\n--- Stage 1: Pre-Fetcher ---")
        resources = ['tenants', 'virtualservice', 'serviceengine']
        results = {}
        for res in resources:
            resp = requests.get(f"{self.base_url}/api/{res}", headers=self.get_headers())
            data = resp.json()
            print(f"Fetched {len(data)} {res}(s).") [cite: 37]
            results[res] = data
        return results

        from framework import AviAutomationFramework
import threading

def run_test_workflow(vs_name):
    fw = AviAutomationFramework()
    fw.login()
    
    # 1. Pre-Fetch
    all_data = fw.pre_fetcher()
    
    # 2. Pre-Validation [cite: 38]
    print("\n--- Stage 2: Pre-Validation ---")
    vs_list = all_data.get('virtualservice', [])
    target_vs = next((vs for vs in vs_list if vs['name'] == vs_name), None)
    
    if target_vs and target_vs.get('enabled') == True: [cite: 39]
        print(f"Validation Passed: {vs_name} is ENABLED.")
        uuid = target_vs['uuid']
        
        # 3. Task/Trigger (Disable VS) [cite: 40]
        print("\n--- Stage 3: Task/Trigger ---")
        put_url = f"{fw.base_url}/api/virtualservice/{uuid}" [cite: 41]
        payload = {"enabled": False} [cite: 44]
        requests.put(put_url, json=payload, headers=fw.get_headers())
        print(f"Sent PUT request to disable {vs_name}.")

        # 4. Post-Validation [cite: 46]
        print("\n--- Stage 4: Post-Validation ---")
        verify = requests.get(put_url, headers=fw.get_headers()).json()
        if verify.get('enabled') == False: [cite: 47]
            print(f"SUCCESS: {vs_name} is now DISABLED.")
    else:
        print("Pre-validation failed or VS not found.")

if __name__ == "__main__":
    # Parallel execution of 2 test cases 
    t1 = threading.Thread(target=run_test_workflow, args=("backend-vs-t1r_1000-1",))
    t2 = threading.Thread(target=run_test_workflow, args=("backend-vs-t1r_1000-1",)) # Repeating for parallel demo
    
    t1.start()
    t2.start()
    t1.join()
    t2.join()

---

# VMware Avi Load Balancer - Test Automation Framework

## Overview

This project is a Python-based modular test automation framework designed to interact with a mock Avi Load Balancer API. The framework demonstrates configuration-driven automation, parallel task execution, and a structured 4-stage test workflow.

## Features

* 
**Configuration-Driven**: All API endpoints, credentials, and test data are managed via `config.yaml`.


* 
**Parallel Execution**: Supports running multiple test cases simultaneously using Python's `threading` module.


* 
**Modular Architecture**: Separates framework logic (API interactions, mock components) from test execution.


* 
**Automated Workflow**: Implements a complete lifecycle: Pre-Fetcher, Pre-Validation, Trigger (Task), and Post-Validation.


* 
**Mock Components**: Includes stubbed methods for SSH and RDP connection validations.



## Project Structure

```text
├── config.yaml          # YAML file for API and test configurations
├── framework.py         # Core framework logic and API methods
├── test_executor.py    # Main script to run the 4-stage workflow
├── requirements.txt     # Python dependencies
└── README.md            # Project documentation

```

## Setup Instructions

### 1. Prerequisites

Ensure you have Python 3.8+ installed.

### 2. Install Dependencies

Install the required libraries using pip:

```bash
pip install -r requirements.txt

```

### 3. Configuration

Update `config.yaml` with your registered credentials:

* 
**URL**: `https://semantic-brandea-banao-dc049ed0.koyeb.app/` 


* 
**Credentials**: Use your registered `username` and `password`.



## Execution Workflow

The framework executes the following stages for the target Virtual Service (`backend-vs-t1r_1000-1`):

1. 
**Pre-Fetcher**: Retrieves all Tenants, Virtual Services, and Service Engines.


2. 
**Pre-Validation**: Checks if the target Virtual Service is currently `"enabled": true`.


3. 
**Task/Trigger**: Sends a `PUT` request to the Virtual Service UUID to set `"enabled": false`.


4. 
**Post-Validation**: Performs a final `GET` request to verify the status change was successful.



## How to Run

To execute the test suite with parallel execution enabled:

```bash
python test_executor.py

```

## Mock API Reference

The framework interacts with the following endpoints:

* 
**POST** `/register`: Initial user registration.


* 
**POST** `/login`: Returns a session token via Basic Auth.


* 
**GET** `/api/virtualservice/<uuid>`: Validates VS status.


* 
**PUT** `/api/virtualservice/<uuid>`: Updates VS configuration.

