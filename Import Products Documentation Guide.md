---


---

<h2 id="product-import-system-developer-documentation">Product Import System Developer Documentation</h2>
<p><strong>Overview</strong></p>
<p>The <strong>Product Import System</strong> provides developers with a robust, scalable way to import products in bulk using CSV or Excel files. This system supports three core operations: <strong>ADD</strong>, <strong>UPDATE</strong>, and <strong>DELETE</strong> products, and is built using microservices with event-driven architecture. The system leverages <strong>Kafka</strong> for asynchronous processing, <strong>Redis</strong> for job coordination, and <strong>Hasura</strong> GraphQL for database communication.</p>
<p>This documentation will guide developers on integrating the product import system, including API endpoints, file validation, error handling, and job status monitoring.</p>
<p><strong>Table of Contents</strong></p>
<ol>
<li>
<p><a href="#system-architecture">System Architecture</a></p>
</li>
<li>
<p><a href="#integration-prerequisites">Integration Prerequisites</a></p>
</li>
<li>
<p><a href="#api-endpoints">API Endpoints</a></p>
</li>
<li>
<p><a href="#import-workflow">Import Workflow</a></p>
</li>
<li>
<p><a href="#file-format-and-validation">File Format and Validation</a></p>
</li>
<li>
<p><a href="#error-handling">Error Handling</a></p>
</li>
<li>
<p><a href="#kafka-and-redis-integration">Kafka and Redis Integration</a></p>
</li>
<li>
<p><a href="#best-practices-for-integration">Best Practices for Integration</a></p>
</li>
</ol>
<p><strong>System Architecture</strong></p>
<p>The <strong>Product Import System</strong> is built with the following components:</p>
<ol>
<li>
<p><strong>Product Import Service</strong>: Handles file uploads, data validation, and dispatches product data for processing.</p>
</li>
<li>
<p><strong>Image Upload Service</strong>: Manages image storage using <strong>MinIO</strong> for product image uploads.</p>
</li>
<li>
<p><strong>Kafka</strong>: Used for event-driven processing of product data in an asynchronous manner.</p>
</li>
<li>
<p><strong>Redis</strong>: Ensures only one worker processes a given product at a time, ensuring data integrity.</p>
</li>
<li>
<p><strong>Hasura GraphQL</strong>: Provides a GraphQL API for interacting with the database and triggering event-driven processes.</p>
</li>
</ol>
<p><strong>Diagram Overview</strong><br>
<img src="" alt="enter image description here"></p>
<p><strong>Integration Prerequisites</strong></p>
<p>To integrate the Product Import System, ensure you have the following set up:</p>
<ol>
<li><strong>Kafka</strong> and <strong>Redis</strong> clusters available for event and job coordination.</li>
<li><strong>MinIO</strong> an object storage service for product images.</li>
<li><strong>Hasura GraphQL</strong> to handle database triggers and real-time processing.</li>
<li><strong>Node.js</strong> (import-consumer microservice) server to handle file uploads and API requests.</li>
</ol>
<p><strong>Environment Variables</strong></p>
<p>KAFKA_BROKER=KAFKA_BROKER<br>
GRAPHQL_URI=“<a href="https://api.spareplace.fr/v1/graphql">https://api.spareplace.fr/v1/graphql</a>”<br>
GRAPHQL_SECRET_HASURA=“ecospare”<br>
SECRET=""</p>
<p>UPLOAD_URL=‘<a href="https://micro.spare-place.com/upload">https://micro.spare-place.com/upload</a>’<br>
URL_ENDPOINT_IMAGE=“<a href="https://images.spare-place.com/spareplace/">https://images.spare-place.com/spareplace/</a>”</p>
<p>REDIS_HOST=“YOUR_REDIS_URL”<br>
REDIS_PORT=“YOUR_REDIS_PORT”<br>
REDIS_PASSWORD=“YOUR_REDIS_PASSWORD”</p>
<p><strong>API Endpoints</strong></p>
<p><strong>1. Upload CSV/Excel File</strong></p>
<ul>
<li><strong>Method</strong>: POST /api/import-products</li>
<li><strong>Description</strong>: Upload a CSV/Excel file for processing product import actions.</li>
<li><strong>Headers</strong>:
<ul>
<li>Content-Type: multipart/form-data</li>
</ul>
</li>
<li><strong>Request</strong>:</li>
</ul>
<pre><code>    curl -X POST "https://yourdomain.com/api/import/upload" \
     -H "Content-Type: multipart/form-data" \
     -F "file=@/path/to/products.csv"
</code></pre>
<ul>
<li><strong>Response</strong>:</li>
</ul>
<pre><code>	{
		"jobId": "12345",
		"message": "File uploaded successfully. Processing has started."
	}
</code></pre>
<p><strong>2. Get Import Job Status</strong></p>
<p><strong>3. Get Job Report</strong></p>
<ul>
<li><strong>Method</strong>: GET /api/import/report/:jobId</li>
<li><strong>Description</strong>: Retrieve the final report for the import job, including the number of successful and failed rows.</li>
<li><strong>Response</strong>:</li>
</ul>
<pre><code>	{
	  "jobId": "12345",
	  "status": "Completed",
	  "report": {
	    "successfulRows": 98,
	    "failedRows": [
	      {
	        "rowNumber": 4,
	        "error": "Missing required field: Product Name"
	      }
	    ]
	  }
	}
</code></pre>
<p><strong>Import Workflow</strong></p>
<ol>
<li>
<p><strong>File Upload</strong>: Users upload a CSV or Excel file containing product data through the /api/import-products.</p>
</li>
<li>
<p><strong>Batch Processing</strong>: The system processes the file in batches to avoid performance bottlenecks. Each product is validated individually before being sent to the database.</p>
</li>
<li>
<p><strong>Asynchronous Processing</strong>: Valid products are dispatched to <strong>Kafka</strong> for further processing. Errors are logged per row.</p>
</li>
<li>
<p><strong>Job Status Monitoring</strong>: Developers can query the import job status via the import_jobs from <strong>HASURA</strong>. Once the job is complete, a detailed report can be fetched.</p>
</li>
</ol>
<p><strong>Example Flow</strong></p>
<ul>
<li><strong>Frontend</strong>: Upload file → Hit /api/import-product API</li>
<li><strong>Backend</strong>: Process file → Dispatch to Kafka → Handle processing in import-consumer microservice + upload images in minio microservice</li>
<li><strong>Backend</strong>: Update job status and batch status for each product</li>
<li><strong>Frontend</strong>: Generate job report</li>
</ul>
<p><strong>File Format and Validation</strong></p>
<p><em><strong>Supported File Formats:</strong></em></p>
<ol>
<li><strong>CSV</strong> (.csv)</li>
<li><strong>Excel</strong> (.xlsx, .xls)</li>
</ol>
<p><em><strong>Required Fields for Each Action</strong></em></p>
<p>In the following link you will find all informations about the validate headers and data:</p>
<ul>
<li><a href="https://docs.google.com/document/d/1peuR8qvMZxNLEjxwgc5dX2KA7hAfHeJuJTdTZ06AK1Q/edit?tab=t.0#heading=h.hiyyazd2tv1c">Product Import Documentation</a></li>
</ul>
<p><strong>Example File</strong></p>
<ul>
<li><a href="https://docs.google.com/spreadsheets/d/1l2hehCujh5c-KC7nWSf9yFhPChUqKFT5-Vojq-b5fjw/edit?gid=0#gid=0">Product Import Template</a></li>
</ul>
<p><strong>Kafka and Redis Integration</strong></p>
<p>The import system uses <strong>Kafka</strong> and <strong>Redis</strong> to handle job coordination and product processing.</p>
<p><strong>Kafka Topics</strong></p>
<ul>
<li><strong>product-import-events</strong>: This topic processes product-related events such as adding, updating or deleting products.</li>
</ul>
<p><strong>Redis for Job Coordination</strong></p>
<p>To ensure that only one worker processes a product at a time, Redis locks are implemented around each product processing task. This prevents race conditions and ensures data consistency.</p>
<p><strong>Best Practices for Integration</strong></p>
<ol>
<li><strong>Batch Size</strong>: Keep file sizes manageable to avoid memory issues on the server. Split large imports into smaller batches.</li>
<li><strong>Monitoring Job Status</strong>: Track job progress and diagnose errors in real-time.</li>
<li><strong>Error Handling</strong>: Always check the job report for detailed error information after a job completes.</li>
<li><strong>Concurrency</strong>: Leverage Kafka and Redis to handle multiple imports concurrently without race conditions.</li>
</ol>
<p>Bugs:</p>
<ul>
<li>Kafka doesn’t logs the latest import-products nor in the console or the pm2 logs.</li>
<li>I updated the code but I don’t see the changes knowing that I stopped and started the server or restart it using <em><strong>pm2 restart</strong></em></li>
<li>Images performance when I import products with images the process slows down.</li>
</ul>

