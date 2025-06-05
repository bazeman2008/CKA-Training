# CKA Training Webapp

A modern, responsive web interface for the CKA (Certified Kubernetes Administrator) training documentation. This webapp provides an intuitive way to navigate through the 5-month training program with a beautiful GUI.

## Features

- 🎨 Modern, responsive design with Kubernetes branding
- 📱 Mobile-friendly interface
- 📚 Interactive documentation browser
- 🗓️ Visual timeline of the 5-month training schedule
- 🔍 Easy navigation between weeks and months
- 📊 Progress tracking visualization
- 🐧 Rocky Linux 8.5 compatibility guide integration

## Quick Start

### Using Docker Compose (Recommended)

1. Navigate to the webapp directory:
   ```bash
   cd webapp
   ```

2. Build and start the container:
   ```bash
   docker-compose up -d
   ```

3. Access the website at:
   - Local: http://localhost
   - Production: http://CKA.forst.fun

### Using Docker Only

1. Build the image:
   ```bash
   docker build -t cka-training-webapp .
   ```

2. Run the container:
   ```bash
   docker run -d -p 80:80 --name cka-webapp cka-training-webapp
   ```

### Development Setup

For local development without Docker:

1. Install a local web server (e.g., Python's built-in server):
   ```bash
   python3 -m http.server 8000
   ```

2. Access at http://localhost:8000

## Directory Structure

```
webapp/
├── index.html          # Main webpage
├── styles.css          # CSS styling
├── script.js           # JavaScript functionality
├── Dockerfile          # Docker image configuration
├── docker-compose.yml  # Docker Compose setup
├── nginx.conf          # Nginx server configuration
└── README.md           # This file
```

## Features Overview

### Homepage Sections

1. **Hero Section**: Eye-catching introduction with key statistics
2. **Training Overview**: Four key benefits and features
3. **5-Month Timeline**: Visual progression through the training program
4. **Weekly Breakdown**: Interactive month-by-month view
5. **Essential Resources**: Quick access to important guides

### Available Endpoints

- `/` - Main homepage
- `/docs/` - Browse documentation files
- `/rocky-linux-guide` - Rocky Linux 8.5 setup guide
- `/testing-strategies` - CKA exam strategies
- `/practice-scenarios` - Hands-on practice scenarios
- `/schedule` - Complete training schedule
- `/health` - Health check endpoint

### Navigation Features

- Smooth scrolling navigation
- Interactive month switching
- Responsive mobile design
- Fixed navigation bar
- Animated scroll effects

## Customization

### Styling

Edit `styles.css` to customize:
- Color scheme (CSS variables in `:root`)
- Typography and fonts
- Layout and spacing
- Animation effects

### Content

Modify `index.html` to update:
- Training statistics
- Section content
- Navigation links
- Footer information

### Functionality

Update `script.js` for:
- Interactive behaviors
- Animation triggers
- Navigation logic
- Dynamic content loading

## Docker Configuration

### Environment Variables

- `NGINX_HOST`: Server hostname (default: CKA.forst.fun)
- `NGINX_PORT`: Server port (default: 80)

### Volume Mounts

The Docker setup automatically mounts:
- Training documentation from parent directory
- Rocky Linux guide
- Testing strategies
- Practice scenarios
- Complete schedule

### Nginx Configuration

The webapp uses a custom Nginx configuration that:
- Serves static files efficiently
- Provides API endpoints for markdown content
- Includes security headers
- Enables gzip compression
- Sets up proper caching

## Production Deployment

### Domain Setup

1. Point `CKA.forst.fun` to your server's IP address
2. Update DNS A record
3. Configure SSL/TLS certificate (recommended)

### SSL/HTTPS Setup

Add SSL configuration to `nginx.conf`:

```nginx
server {
    listen 443 ssl;
    server_name CKA.forst.fun;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    # Your existing configuration
}
```

### Using with Reverse Proxy

If using with Traefik or another reverse proxy, the Docker Compose file includes Traefik labels for automatic configuration.

## Monitoring and Logs

### View Logs

```bash
# Docker Compose logs
docker-compose logs -f cka-webapp

# Nginx access logs
docker exec cka-training-webapp tail -f /var/log/nginx/access.log
```

### Health Check

The webapp includes a health endpoint:
```bash
curl http://CKA.forst.fun/health
```

## Troubleshooting

### Common Issues

1. **Port 80 already in use**:
   ```bash
   # Change port in docker-compose.yml
   ports:
     - "8080:80"
   ```

2. **Documentation not loading**:
   - Verify volume mounts in docker-compose.yml
   - Check file permissions

3. **Styling issues**:
   - Clear browser cache
   - Check for CSS/JS errors in browser console

### Performance Optimization

- Enable Nginx gzip compression (already configured)
- Use CDN for static assets in production
- Optimize images and fonts
- Implement browser caching headers

## Contributing

To add new features or modify the webapp:

1. Make changes to HTML, CSS, or JS files
2. Test locally
3. Rebuild Docker image
4. Deploy updated container

## License

This webapp is part of the CKA Training repository and follows the same licensing terms.