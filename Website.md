# Create a web interface to display some videos


---
The objective of our project is to create a High Available (HA) cluster on some Raspberry Pi using k3s and the aim of this cluster to 
deploy a video streaming platform.

This tutorial is made for all people who want to set up a platform displaying videos. It is really basic and the aim is to have something working properly before implementing other functionalities.

First, we developed a local website using a local server and in a second part, we dockerized our content in order to deploy on the cluster.

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

We use Xampp to set up a local server that you can download here: [Xampp](https://www.apachefriends.org/fr/download.html).

It is really useful to test, without too much convenience, our website and its functionalities.

## Let's start!

### Requirements

- A computer with an installed OS
- Xampp software installed
- An IDE to develop your site (we used VSCode in our case)

### First Step: Develop the structure of the site

It is necessary to code, but, since we are not professionals designers, we used a basic bootstrap template to have a simple and aestethic website first page. 

We added a navbar with these elements: logo, home, upload, videos and connection but we'll come back later to the upload and connection parts.

In the main page, we wanted to have a beautiful animated background at the top and then, a section where all videos are sorted.
Finally, we added a small footer to the website with the name of the project.

### Second Step: Streaming video
We saw previously the template of our site. It is time to implement more functionalities!

Our project is to display videos but first, we have to encode them in different resolutions (audio and video) in order to have a single file (a .mpd one) that will allow the stream of the video. 

To do so, we implemented an upload button linked to a file called upload.php that manage the operations. 
We are going to develop these operations below:

We have to download these two softwares:
- FFMPEG (that you can find here: [Ffmpeg](https://www.ffmpeg.org/download.html))
- GPAC (also available here: [Gpac](https://gpac.wp.imt.fr/downloads/))

Extract the downloaded ffmpeg zip file to 
  "c:\ffmpeg"
  Navigate to the "bin" folder under c:\ffmpeg and copy the address using Ctrl+C
Open up the System information window. (You can use the shortcut WindowsKey+Break/Pause)
Click "Advanced system settings"
Click "Environment Variables..."
Select the "Path" variable under System variables
Click "Edit..."
Click "New"
Type Ctrl+V to paste in the address where you extracted ffmpeg to earlier 
  
### Third Step: Upload and database
