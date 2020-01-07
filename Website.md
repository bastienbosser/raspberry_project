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

It is necessary to code, but, since we are not professionals designers, we used a basic bootstrap template to have a simple and aestethic website first page. You can see the whole documentation of the template here : [Template](https://demos.creative-tim.com/material-kit/docs/2.1/getting-started/introduction.html)

We added a navbar with these elements: logo, home, upload, videos and connection but we'll come back later to the upload and connection parts.

In the main page, we wanted to have a beautiful animated background at the top and then, a section where all videos are sorted.
Finally, we added a small footer to the website with the name of the project.

### Second Step: Streaming video
We saw previously the template of our site. It is time to implement more functionalities!

Our project is to display videos but first, we have to encode them in different resolutions (audio and video) in order to have a single file (a .mpd one) that will allow the stream of the video. 

We are going to develop the operations below:

We have to download these two softwares:
- FFMPEG (that you can find here: [Ffmpeg](https://www.ffmpeg.org/download.html))
- GPAC (also available here: [Gpac](https://gpac.wp.imt.fr/downloads/))

Once it is done, do the following steps:
   
    Extract the downloaded ffmpeg zip file to "c:\ffmpeg"
    Navigate to the "bin" folder under c:\ffmpeg and copy the address using Ctrl+C
    Open up the System information window. (You can use the shortcut WindowsKey+Break/Pause)
    Click "Advanced system settings"
    Click "Environment Variables..."
    Select the "Path" variable under System variables
    Click "Edit..."
    Click "New"
    Type Ctrl+V to paste in the address where you extracted ffmpeg to earlier 
    
Then, we need to test the programs:

    Open the Run dialog using the shortcut WindowsKey+R
    Type in "cmd" into the textbox and press ENTER
    In the command prompt type "ffmpeg" and press ENTER
    If you see a bunch of text, like the first image, you have set up ffmpeg properly!
    Type "cls" to clear the screen
    Type "mp4box" and press ENTER
    If you see a bunch of text, like the second image, you have set up mp4box successfully!
    If you see something like "ffmpeg" is not recognized as an internal or external command... then something is not setup correctly. You need to go back to step 1 and reinstall the programs.
    
To continue, it is necessary to download an .avi file of your choice. Try to execute the following command in the command prompt:

    ffmpeg -i input.avi -s 160x90 -c:v libx264 -b:v 250k -g 90 -an input_video_160x90_250k.mp4
    
Here are what the arguments mean:

    -i input.avi - tells ffmpeg where the input file is
    -s 160x90 - is the resolution we want to encode our input file to
    -c:v libx264 - specifies the audio codec to use, in this case we want h264
    -b:v 250k - is the bitrate we want to encode the video to
    -g 90 - tells ffmpeg we want a keyframe interval (GOP length) of 90
    -an - do not encode audio
    input_video_160x90_250k.mp4 - the output file
    
Now, encode the audio using:

    ffmpeg -i input.avi -c:a aac -b:a 128k -vn input_audio_128k.mp4 
    
Here are what the arguments mean:

    -i input.avi - the input file
    -c:a - the audio codec
    -b:a - the bitrate of the audio
    -vn - do not encode video
    input_audio_128k.mp4 - the output file
    
You have successfully created an encoded audio file and encoded video files, now you need to run the following commands in order to create all the necessary files for the rest of the Instructable.

    ffmpeg -i input.avi -s 160x90 -c:v libx264 -b:v 250k -g 90 -an input_video_160x90_250k.mp4
    ffmpeg -i input.avi -s 320x180 -c:v libx264 -b:v 500k -g 90 -an input_video_320x180_500k.mp4
    ffmpeg -i input.avi -s 640x360 -c:v libx264 -b:v 750k -g 90 -an input_video_640x360_750k.mp4
    ffmpeg -i input.avi -s 640x360 -c:v libx264 -b:v 1000k -g 90 -an input_video_640x360_1000k.mp4
    ffmpeg -i input.avi -s 1280x720 -c:v libx264 -b:v 1500k -g 90 -an input_video_1280x720_1500k.mp4
    ffmpeg -i input.avi -c:a aac -b:a 128k -vn input_audio_128k.mp4
    
Now that you have all you encoded files, you can turn them into DASH compatible files. This process will generate MPEG-4 initialization files that the DASH player reads at load time and a manifest file that tells the player where all the necessary files are and how to read them.

To prepare your files for streaming you need to use the following command:

    mp4box -dash 5000 -rap -profile dashavc264:onDemand -mpd-title BBB -out manifest.mpd -frag 2000 input_audio_128k.mp4 input_video_160x90_250k.mp4 input_video_320x180_500k.mp4 input_video_640x360_750k.mp4 input_video_640x360_1000k.mp4 input_video_1280x720_1500k.mp4
    
    -dash 5000 - cuts the input files into 5 second segments
    -rap - forces the segments to start with random access points. In other words the allows for seeking of the video
    -profile dashavc264:onDemand - use the onDemand profile (you can look in the dash specifications to find out more information about the different kinds of profiles)
    -mpd-title - sets the title of the manifest to "BBB"
    -out - the output file name
    -frag - sets the fragment length to 2 seconds. This must be less than the value specified with -dash
    
The process is now done and if you need further information, click here: [Mpeg-Dash](https://www.instructables.com/id/Making-Your-Own-Simple-DASH-MPEG-Server-Windows-10/?fbclid=IwAR0vAzRS--Jg7V9TS25tiYMLQGHmv0i87S1O0zJdX2Pabj0inWvZAKtqeM4). Now, we just need a player able to read the .mpd file to see our video.

After few researches, we just found one called Dash JavaScript Player.

Create a video element somewhere in your html and provide the path to your mpd file as src. Also ensure that your video element has the data-dashjs-player attribute on it.

      <video data-dashjs-player autoplay src="https://dash.akamaized.net/envivio/EnvivioDash3/manifest.mpd" controls>
      </video>

Add dash.all.min.js to the end of the body.

      <body>
         ...
      <script src="yourPathToDash/dash.all.min.js"></script>
      </body>

And the process is finished. You could see your video now! If you need further information, click here : [DashPlayer](https://github.com/Dash-Industry-Forum/dash.js) .
  
### Third Step: Upload

We wanted to bring an additional feature: upload a video and encode it directly. To do so, we implemented an upload button linked to a file called upload.php that manage the operations. 

Indeed, we found a useful php function:

       exec ( string $command [, array &$output [, int &$return_var ]] ) : string
       
This way, we can execute the same command lines to encode the video automatically after uploading a video.

Then, we need to verify few things shown below:

      // Check if file already exists
      if (file_exists($target_file)) {
         echo "Sorry, file already exists.";
         $uploadOk = 2;
         header('location:template.php?msg=3');
         exit();
      }

      // Allow certain file formats
      if($imageFileType != "avi" && $imageFileType != "mp4" && $imageFileType != "wmv") {
         echo "Sorry, only AVI, MP4, WMV files are allowed.";
         $uploadOk = 0;
         header('location:template.php?msg=2');
         exit();
      }

      // Check if $uploadOk is set to 0 by an error
      if ($uploadOk == 0) {
         echo "Sorry, your file was not uploaded.";
         header('location:template.php?msg=2');
      }  
      
If you need further information, click here : [PHP exec](https://www.php.net/manual/fr/function.exec.php).

The upload feature is done except one element: if you are running on a xampp server, you might have a problem with the php settings so you need to change these ones on php.ini in order to have no problem uploading and encoding your file:

    max_execution_time=600
    post_max_size=2048M


### Fourth Step: Export our project through Docker

