# Study Notes: FastAPI, REST APIs, and Machine Learning Deployment

---

## 1. Synchronous vs. Asynchronous APIs in FastAPI

### Core Concepts

* **Synchronous API (Blocking):** Requests are handled sequentially. When a thread encounters an I/O operation (database query, third-party API call), it blocks and waits idly until completion. Defined in FastAPI using standard `def`.
* **Asynchronous API (Non-Blocking):** When an I/O operation is hit, the task pauses, yields control back to the event loop, and frees up the thread to handle other requests. Defined using `async def` and `await`.

### Pros and Cons Comparison

| Feature | Synchronous APIs (`def`) | Asynchronous APIs (`async def`) |
| --- | --- | --- |
| **Concurrency** | **Low to Medium:** Consumes dedicated threads per active request; queues up when threads run out. | **High:** Handles thousands of concurrent connections using an event loop via a single thread (for I/O-bound tasks). |
| **Simplicity** | **Very High:** Linear, easy to write and debug. | **Medium/Low:** Requires `async/await` syntax and async-compatible libraries. |
| **I/O-Bound Performance** | **Poor under load:** Threads spend time waiting idly. | **Excellent:** Maximizes efficiency for I/O-heavy operations. |
| **CPU-Bound Performance** | **Decent, but limited:** Heavy tasks block the thread. | **Bottlenecks easily:** Unoffloaded CPU-heavy code freezes the entire event loop. |
| **Ecosystem** | Virtually **all** Python libraries work natively (e.g., `requests`, standard DB drivers). | Requires **async equivalents** (e.g., `httpx`, `asyncpg`). Blocking libraries ruin async performance. |

---

## 2. FastAPI Handling & Rules of Thumb

* **`def` (Sync):** FastAPI automatically executes these in an external thread pool (via `anyio`/Starlette). This prevents blocking synchronous code from freezing the whole server application.
* **`async def` (Async):** Runs directly on the main event loop. Blazing fast for I/O-bound tasks, provided async clients/drivers are used.

---

## 3. Deploying Machine Learning / Deep Learning Models

### The Golden Rule: Why Synchronous (`def`) is Preferred for ML

Machine learning inference and deep learning forward passes are **CPU/GPU-bound workloads** (heavy matrix multiplications and tensor math).

#### Technical Reasons:

1. **The Event Loop is Single-Threaded:** Python’s `asyncio` runs on a single thread using cooperative multitasking. Libraries like PyTorch, TensorFlow, NumPy, and Scikit-learn execute core math in compiled C/C++ or CUDA code and **do not contain `await` points**.
2. **Blocking the Event Loop:** If you put heavy model inference inside an `async def` function, Python’s main thread must sit and wait for the C++ layer to finish. This **freezes the entire event loop**, halting all other incoming requests.
3. **FastAPI's Thread Pool Offloading (`def`):** When you use a standard `def` endpoint, FastAPI offloads the execution to an external thread pool. While **Thread B** computes the model prediction, the main event loop thread remains completely free to accept and serve new incoming requests.

---

### Best Practices for ML/DL API Architecture

* **Load the Model Once at Startup:** Never load your model inside the endpoint function, or it will reload from disk on every single request, spiking latency and memory usage. Use global variables or FastAPI lifespan events.
* **Separate Real-Time Inference from Heavy Batch Jobs:** Keep the API focused strictly on real-time predictions. Use task queues (like Celery or ARQ) with message brokers (like Redis) for long-running batch predictions or model training.
* **Production ASGI Server Configuration:** Run your application using a production server like **Uvicorn** managed by **Gunicorn** across multiple worker processes to utilize multiple CPU cores:
```bash
gunicorn main:app --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000

```


*(Note: Each worker loads its own model copy into RAM; ensure your system memory can handle the total footprint).*
* **Containerization:** Always use **Docker** and `docker-compose` to guarantee consistent environments across local development and production servers.
* **Specialized Serving Engines (For Deep Learning):** For heavy models like Large Language Models (LLMs) or complex computer vision architectures, consider specialized model serving engines (e.g., **vLLM**, **Triton Inference Server**, **TorchServe**) in front of or alongside your FastAPI application for advanced dynamic batching and hardware optimization.