Choosing the Base Image
=======================

For a Jenkins inbound agent, a common starting point is:

FROM jenkins/inbound-agent:latest

Why?

It already contains:

Java
Jenkins remoting agent
Required connection components

You only need to add your DevOps tools.
