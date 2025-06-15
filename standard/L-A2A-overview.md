
## **Localized A2A (L-A2A) Specification**

This specification outlines a protocol for communication and interoperability between agentic components within a single application or process. It is a localized adaptation of the standard A2A protocol, with all network-based interactions replaced by local function calls.

### **1\. Core Concepts**

The L-A2A protocol is built around the following core concepts:

* **Agents:** These are the fundamental actors in the system. In the L-A2A context, an "agent" can be a class, a module, or any distinct component within an application that can both initiate and handle tasks.  
* **Tasks:** A **Task** represents a unit of work to be performed. It has a lifecycle (e.g., pending, running, completed, failed) and is the central object around which communication is organized.  
* **Messages:** Agents communicate by exchanging **Messages** within the context of a Task. A message contains the actual content or data being passed between agents.  
* **Artifacts:** An **Artifact** represents the output or result of a completed Task.

### **2\. L-A2A Architecture**

The L-A2A protocol replaces the client-server, network-based architecture of the original A2A with a local, event-driven, or direct-call architecture.

#### **2.1. Agent Registry (Replaces Network Discovery)**

Instead of discovering agents via network endpoints (URLs for Agent Cards), L-A2A uses a local **Agent Registry**.

* **Implementation:** The Agent Registry can be implemented as a singleton class, a global dictionary, or a dedicated service within your application.  
* **Registration:** On application startup, each agent registers itself with the Agent Registry, providing its "Agent Card" as a local data structure (e.g., a dictionary or an object). This Agent Card contains:  
  * A unique agent ID (e.g., a string name).  
  * A list of capabilities or supported task types.  
  * A reference to the agent's handler function or object.  
* **Discovery:** When an agent (the "caller") wants to initiate a task with another agent (the "callee"), it queries the Agent Registry to get the callee's information and handler reference.

#### **2.2. Communication (Replaces HTTP and JSON-RPC over Network)**

All communication is done via direct function calls. The JSON-RPC 2.0 message format is still used for the data passed to these functions to maintain structural consistency with the original protocol.

* **Caller Agent:**  
  1. Constructs a JSON-RPC 2.0 compliant request object (as a dictionary or string).  
  2. Looks up the callee agent in the Agent Registry.  
  3. Calls the callee's handler function directly, passing the request object as an argument.  
* **Callee Agent:**  
  1. The handler function receives the request object.  
  2. Parses the request and executes the task.  
  3. Returns a response object, also in JSON-RPC 2.0 format.

#### **2.3. Streaming and Asynchronous Operations (Replaces SSE)**

For long-running tasks that require streaming updates, Server-Sent Events (SSE) are replaced with a local publish-subscribe (pub/sub) mechanism or callbacks.

* **Implementation:** A simple event emitter or a more robust message bus can be used.  
* **Flow:**  
  1. When a caller initiates a task that supports streaming, it also provides a callback function or subscribes to a specific task ID in the event system.  
  2. As the callee agent processes the task, it emits status updates (e.g., TaskStatusUpdateEvent, TaskArtifactUpdateEvent).  
  3. The caller's callback function is invoked for each event, receiving the update data.

### **3\. Example Implementation (Conceptual Python Code)**

Here is a simplified, conceptual example in Python to illustrate the L-A2A standard:  
`# 1. Agent Registry`  
`AGENT_REGISTRY = {}`

`def register_agent(agent_id, capabilities, handler):`  
    `"""Registers an agent in the local registry."""`  
    `AGENT_REGISTRY[agent_id] = {`  
        `"capabilities": capabilities,`  
        `"handler": handler,`  
    `}`

`# 2. Event System (for streaming)`  
`EVENT_BUS = {} # Simple dict-based event bus`

`def subscribe(task_id, callback):`  
    `"""Subscribes to updates for a given task."""`  
    `if task_id not in EVENT_BUS:`  
        `EVENT_BUS[task_id] = []`  
    `EVENT_BUS[task_id].append(callback)`

`def publish(task_id, event_data):`  
    `"""Publishes an update for a task."""`  
    `if task_id in EVENT_BUS:`  
        `for callback in EVENT_BUS[task_id]:`  
            `callback(event_data)`

`# 3. Agent Definitions`  
`def data_processor_agent_handler(request):`  
    `"""A simple agent that processes data."""`  
    `task_id = request["params"]["task_id"]`  
    `publish(task_id, {"status": "running"})`  
      
    `# ... processing logic ...`  
    `result = {"data": "processed data"}`  
    `publish(task_id, {"status": "completed", "artifacts": [result]})`  
      
    `return {"jsonrpc": "2.0", "result": "Task completed", "id": request["id"]}`

`# 4. Main Application Logic`  
`if __name__ == "__main__":`  
    `# Register agents on startup`  
    `register_agent(`  
        `"data_processor",`   
        `capabilities=["process_data"],`   
        `handler=data_processor_agent_handler`  
    `)`

    `# --- Caller Agent Logic ---`  
    `callee_id = "data_processor"`  
    `task_id = "task_123"`

    `def my_callback(event):`  
        `"""Callback to handle streaming updates."""`  
        `print(f"Received update: {event}")`

    `# Subscribe to updates for our task`  
    `subscribe(task_id, my_callback)`

    `# Discover and call the agent`  
    `if callee_id in AGENT_REGISTRY:`  
        `handler = AGENT_REGISTRY[callee_id]["handler"]`  
          
        `# Create a JSON-RPC-like request`  
        `request = {`  
            `"jsonrpc": "2.0",`  
            `"method": "tasks/send",`  
            `"params": {`  
                `"task_id": task_id,`  
                `"message": "Please process this data.",`  
            `},`  
            `"id": 1,`  
        `}`  
          
        `# Make the local "A2A" call`  
        `response = handler(request)`  
        `print(f"Final response: {response}")`

### **4\. Authentication and Security**

In this local context, complex authentication mechanisms (like OAuth tokens in HTTP headers) are unnecessary. Security is handled by the application's own architecture and code-level access controls. You can ensure that only specific modules or classes are allowed to call certain agent handlers.  
By following this L-A2A specification, you can leverage the structured communication patterns of the A2A protocol to build robust, interoperable agentic systems within a single application, without the overhead and complexity of network communication.