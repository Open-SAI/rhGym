# Red Hat Gym 
A pragmatic space to save notes, tips, commands in a Red Hat Engineer training journey...

### Git / github 
- Init local new repo ```$ git init```
### Podman **web.sh** 

Bash script used to resize a group o images to be uploaded to the web:
- must be executed in the folder that contains the images
- must be parameterized (edit the bash script):
    - IMG_FORMAT=jpeg > what type of image do you convert to web size?
    - IMG_COUNTER=12 > if is not equal to zero, the new images will be incrementally named 1.jpg, 2.jpg... etc
    - IMG_SIZE=1600x1200 > size in pixels of the new image
    - IMG_QUALITY=70 > quality percentage of the new image vs. the original
    - CLEAN_FILE_NAMES=1 > use to clean file name with spaces
- the script use the tool [detox](https://detox.sourceforge.net/) to clean the file names, available in the standard repos of the main linux distros
