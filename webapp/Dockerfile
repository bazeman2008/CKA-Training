# Use nginx alpine image for lightweight deployment
FROM nginx:alpine

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy website files to nginx html directory
COPY index.html /usr/share/nginx/html/
COPY styles.css /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/

# Copy markdown files and convert them to HTML (we'll use a simple approach)
COPY ../5-Month-Study-Plan /usr/share/nginx/html/docs/
COPY ../Rocky-Linux-8.5-Setup-Guide.md /usr/share/nginx/html/rocky-linux-guide.md
COPY ../CKA-Testing-Strategies.md /usr/share/nginx/html/testing-strategies.md
COPY ../PracticalSenarioExmp.md /usr/share/nginx/html/practice-scenarios.md
COPY ../CKA-Schedule.md /usr/share/nginx/html/schedule.md

# Create API endpoint for serving markdown content
RUN mkdir -p /usr/share/nginx/html/api

# Set proper permissions
RUN chown -R nginx:nginx /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]