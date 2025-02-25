# Storage in Docker
## File system

![image](https://github.com/user-attachments/assets/748b443c-d41e-41c3-8ecc-ab270e18c788)

## Layered architecture
When Docker builds images, it builds these in a layered architecture. Each line of instruction in the Docker file creates a new layer in the Docker image with just the changes from the previous layer. 

![image](https://github.com/user-attachments/assets/af05c2d6-0c50-4e7a-8c23-b371e25943f9)

You can see when build images which have changes, Docker is not going to build the first three layers. Instead, it reuses the same three layers it built for the first application frm cache. And only creates the last two layers with the new sources and the new entrypoint. This way Docker builds images faster and efficiently saves disc space. Thus saving us a lot of time during rebuilds and updates.

![image](https://github.com/user-attachments/assets/131191ef-63fc-4477-a570-868c603b4743)

If i were to log into the newly created container and say create a new file called temp.text, it will create that file in the container layer which is read and write. 

![image](https://github.com/user-attachments/assets/176969e3-9246-450f-b64b-c7568d8dc0a2)

The image layer being read only just means that the files in these layers will not be modified in the image itself. So the image will remain the same all the time until you rebuild the image using the docker build command.

All of the data that was stored in the container layer also gets deleted. The change we made to the app.py and the new temp file we created will also get removed.

## Volumes

![image](https://github.com/user-attachments/assets/cc50b4aa-74bc-4c6b-bcf8-d5f77071c545)

## Storage drivers

![image](https://github.com/user-attachments/assets/4579ef8b-1d69-448f-b4f8-9791fe727630)

## Volume Driver Plugins in Docker

![image](https://github.com/user-attachments/assets/07591d70-1b19-486b-9d6d-c20071b563ca)




