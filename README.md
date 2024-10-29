# PHP and Nginx Dockerized Application

direct to directory and run `docker-compose up -d` and visit `http://localhost:8080`
thats it your index.php files is serving from docker

## Step-by-Step Request Flow

### 1. Request Reaches Nginx (Port 443)

- When you visit `https://localhost:8443`, the request is directed to the **Nginx container** because `docker-compose.yml` maps `8443` (on your local machine) to `443` (within the Nginx container).
- Nginx listens on port `443` for HTTPS traffic. It then reads the incoming request, checks the requested file type, and decides whether it can handle the request directly or needs to pass it to PHP.

### 2. Nginx Checks if the Request Is for a Static File

- If the request is for a static file (like `.css`, `.js`, or images), Nginx serves the file directly from the `root` directory (`/var/www/html`).
  - **Example**: A request for `https://localhost:8443/style.css` would be served directly from `/var/www/html/style.css` without involving PHP.

### 3. Nginx Detects a PHP File Request

- When the request is for a PHP file (such as `index.php`), Nginx identifies this based on the `.php` extension.
- Nginx forwards the PHP request to the **PHP-FPM** service (running in the `app` container) for processing.

### 4. Nginx Passes the Request to PHP-FPM

- **FastCGI**: Nginx communicates with PHP-FPM via the FastCGI protocol, which is optimized for handling requests between web servers and backend processors.
- The request is forwarded to PHP-FPM at `fastcgi_pass app:9000`:
  - `app` references the `app` service/container running PHP-FPM.
  - `9000` is the port where PHP-FPM listens for incoming requests.

### 5. PHP-FPM Processes the PHP File

- PHP-FPM receives the request and processes the requested PHP file (e.g., `index.php`).
  - For example, if `index.php` includes `echo "Hello World";`, PHP-FPM executes this line and generates the response.
- PHP returns the processed output (usually HTML or plain text) to Nginx.

### 6. Nginx Sends the Response to the Browser

- Nginx receives the output from PHP-FPM, then forwards it to the client (your browser).
- The browser displays the response, which, in this case, shows “Hello World” as output.

## Summary of Nginx and PHP-FPM Roles

- **Nginx**:

  - Manages incoming HTTP/HTTPS requests.
  - Serves static files directly from the `root` directory.
  - Forwards requests for PHP files to PHP-FPM via FastCGI.

- **PHP-FPM**:
  - Handles PHP file processing only.
  - Receives requests from Nginx, executes PHP code, and returns the output back to Nginx.

## Why Use This Architecture?

- **Performance**: Nginx handles static files directly, reducing load on PHP and enhancing performance.
- **Security**: Nginx manages HTTPS, so PHP doesn’t handle SSL/TLS directly.
- **Modularity**: Nginx and PHP-FPM can scale independently. If PHP processing becomes a bottleneck, you can add more PHP-FPM instances without altering Nginx configuration.

This setup leverages each service's strengths to efficiently serve your application.

## Generate a Self-Signed SSL Certificate

```
mkdir certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout certs/server.key -out certs/server.crt
```
