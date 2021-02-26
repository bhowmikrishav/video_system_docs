# VIDEO PROCESSING DAEMON CRONJOB

INTRODUCTION:- Cron is a utility that schedules a command or script on your server to run automatically at a specified time and date. A cron job is the scheduled task itself. Cron jobs can be very useful to automate repetitive tasks.Cron Jobs are used for scheduling tasks to run on the server. They're most commonly used for automating system maintenance or administration. However, they are also relevant to web application development. There are many situations when a web application may need certain tasks to run periodically

PROCESSING CRON JOBS:- The cron daemon is a long-running process that executes commands at specific dates and times. You can use this to schedule activities, either as one-time events or as recurring tasks. To schedule one-time only tasks with cron, use the at or batch command. Data processing is most often triggered by user actions, either your end-user or internal users: back-office, admin superusers, or support users maybe. In some cases though, it may happen that some data processing needs to happen on its own, following a schedule rather than being triggered by direct user activity on the product.

LOOKUP MECHANISM TO FIND A PROCESSING JOB :- The look-up mechanism helps find a job , find a video which require processing in multiple resolutions. We perform a query on the database to search for unprocessed resolution of video upload. 

MECHANISM TO LOAD FILE LOCALLY :- First, We clear the temporary folder then load the raw video file in the temporary folder . const raw_file_name = "temp/raw"+mime_to_ext(file_manifest.mime_type) . Mime_to_exe is a function which makes sure whether the video file is in the desrired format.

PROCESSING AND MANIFEST CREATION :- When a video stream is set up, a video manifest file is delivered to the player, itemizing the available streams. Like a restaurant menu, the manifest tells the player what resolutions and bitrates are available, and the player chooses an appropriate stream. 


