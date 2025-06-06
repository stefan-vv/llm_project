# LLM basic project structure

- RAG would be an appropriate approach which allows supplying context to the selected model from externally provided sources
- RAG is modular and the underlying generative model can be changed
- The "context" documents can be stored in a vector database, which the model should access
- The context reduces the probability of model hallucinations

### Basic Components:

- GenAI Model
- DB1 - RAG vector database
- DB2 - Database that should store information retrieved from third-party APIs (e.g. jira), mongoDB might be a good choice
- model API keys (env vars)

### Basic Process

- Receiver prompt request
- Read data from jira API
  - if jira data for given user is missing, terminate the process early
- Load context data
- Embed context data
- Store context data in vector DB
- At query time, retrieve the most relevant chunks
- Pass the retrieved data into the selected LLM model
- Return the provided output

## Overview

1. `llm_api/infrastructure/server/llm_api_server.py` is the starting point, a server
    - if there is a UI, the UI would send requests to the server, FastAPI could be used here
    - if there is no UI, direct requests can be sent to the endpoint
    - a more simplistic approach could be used for direct prompting
2. The request to the server should have the prompt as payload and send request via the jira API from the controller `llm_api/jira/jira_controller.py`
3. When the data is retrieved it should preferably be stored in a DB using `llm_api/jira/jira_repository.py`. If the data is missing for the given user, the process should end here.
4. From here `llm_api/llm_infra/context_data_loader/context_data_loader.py` should get documents/data that provides context to the model (preferably from cloud storage)
5. Using `llm_api/llm_infra/embedder/context_data_embedder.py` the context data should be embedded and stored in a vector DB using `llm_api/llm_infra/context_data_retriever/context_data_retriever_controller.py`
6. Using `llm_api/llm_infra/embedder/prompt_query_embedder.py` the promt/user query should be embedded
7. Vector db search using the embedded user query/prompt using `llm_api/llm_infra/context_data_retriever/context_data_retriever_controller.py` should yield the context information
8. Using the`llm_api/llm_infra/model/model_api_controller.py` the model is called with the provided context and prompt, yielding the model output
9. The server send back the output or prints it
