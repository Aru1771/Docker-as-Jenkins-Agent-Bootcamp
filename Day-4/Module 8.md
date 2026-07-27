Best Practices
===============
Use a stable base image

Avoid floating versions in production when reproducibility matters.

Install only required tools

Don't install software you don't use.

Keep the image updated

Regularly rebuild to include:

Security patches
Tool updates
Base image fixes
Run as a non-root user

Avoid running builds as root unless absolutely necessary.

Keep the image small
Remove package caches
Combine RUN commands where appropriate
Avoid unnecessary packages
