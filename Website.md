# Create a web interface to display some videos


---
The objective of our project is to create a High Available (HA) cluster on some Raspberry Pi using k3s and the aim of this cluster to 
deploy a video streaming platform.

This tutorial is made for all people who want to set up a platform displaying videos. It is really basic and the aim is to have something working properly before implementing other functionalities.

First, we developed a local website using a local server and in a second part, we dockerized our content in order to deploy in on the cluster.

Our method is probably not the best, but it works properly. We are not perfect, we
made a lot of mistakes to come to a decent result. If you have a better
solution for that setup, please send us reports and feedbacks.

---

## Who are we?

This solution is proposed by a group of students in the CyberSecurity Classroom (2019)
of the Engineering School called "ISEN Yncrea Méditerranée" at Toulon (France).
The team is composed of BOSC Virginie, BOSSER Bastien, RIGAUX Thomas, SODA Théo and VALLAURI Mélissa.
This whole tutorial is the result of our project.

## Why do we use a local website and server?

We use Xampp to set up a local server that you can download here: X
It is really useful to test, without too much convenience, our website and its functionalities.

## Let's start!

### Requirements

- A computer with an installed OS
- Xampp software installed
- An IDE to develop your site

### First Step: Develop the structure of the site

It is necessary to code, but, since we are not professionals designers, we used a basic bootstrap template to have a simple and aestethic website first page. 
We added a navbar with these elements: logo, home, upload, videos and connection but we'll come back later to the upload and connection parts.
In the main page, we wanted to have a beautiful animated background at the top and then, a section where all videos are sorted.
Finally, we added a small footer to the website with the name of the project.

### Second Step: Upload and database
We saw previously the template of our site. It is time to implement more functionalities!

