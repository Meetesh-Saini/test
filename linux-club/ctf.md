# How to Submit a Dynamic CTF?

I will be using an example of a simple base64 challenge.

## Make a CTF
Make a CTF of your choice with usual categories:

- Web exploitation
- Cryptography
- Binary exploitation
- Reverse engineering
- Forensics
- OSINT
- Scripting based
- Miscellaneous

Be creative and combine two or more categories if needed.
Let say I create a simple base64 challenge in which user has to decode the base64 to get the flag. First I need to encode the flag.
There can be two ways to do this:
- Use online tools (do not use this)
- Use scripting (use this instead)

## Write a script to make the challenge

Write the script using your preferred language (Bash, Python, JavaScript, or other)

For my base64 challenge, I will use `bash`
```bash
#!/bin/bash 

printf "flag_{s3cr3T}" | base64 > challenge.txt
```

OR I can use `python`
```python 
#!/usr/bin/env python
import base64

flag_bytes = "flag_{s3cr3T}".encode()
base64_bytes = base64.b64encode(flag_bytes)
base64_string = base64_bytes.decode()

with open("challenge.txt", "w") as f:
    f.write(base64_string)
```

OR I can use `javascript`
```js
#!/usr/bin/env node

const fs = require('fs');

flag = "flag_{s3cr3T}"
base64 = btoa(flag)
fs.writeFileSync("challenge.txt", base64);
```

In this example the **flag was static.**

## Make the script dynamic

Replace the flag with the first command line argument passed.

The new scripts will be:

- Dynamic Bash script
```bash
#!/bin/bash 

flag=$1 # $1 means the first cli argument
printf $flag | base64 > challenge.txt
```

- Dynamic Python script
```python 
#!/usr/bin/env python
import base64
import sys 

flag = sys.argv[1] # sys.argv[1] is the first cli argument
flag_bytes = flag.encode()
base64_bytes = base64.b64encode(flag_bytes)
base64_string = base64_bytes.decode()

with open("challenge.txt", "w") as f:
    f.write(base64_string)
```

- Dynamic JavaScript script
```js
#!/usr/bin/env node

const fs = require('fs');

flag = process.argv[2] // In node the 0 and 1 index will be `node` and filename so the first cli argument will be at index 2
base64 = btoa(flag)
fs.writeFileSync("challenge.txt", base64);
```

## Make the Dockerfile
Create the Dockerfile to make the challenge's docker image.

I will be making the docker image for `python` script of our base64 challenge as an example.

- Choose the base image
```dockerfile
FROM python:slim
```
- Set the working directory
```dockerfile
WORKDIR /root 
```
- Install dependencies
In this example we do not have any external dependency but if the challenge has external dependencies like PIL for image manipulation, numpy or other tools like gcc, MongoDB, sqlite3, and more.
```dockerfile
# If the dependency is in package managers
RUN apt-get update && apt-get install -y gcc
RUN pip install pillow

# If the dependency is from a GitHub repo 
# For example if I want to download source code of https://github.com/Meetesh-Saini/sha256 repo's main branch, I would run the following command
RUN wget https://github.com/Meetesh-Saini/sha256/archive/main.zip -O out.zip && unzip out.zip
```
- Configure the environment
```dockerfile
# Set any environment variable if required
ENV MONGO_URI=...
ENV REDIS_DB=...

# Set ssh if the challenge requires it
# Expose the required ports
EXPOSE 80 22
```

- Copy the challenge related files in `/root`
```dockerfile
# The following line copies all of the files present in the current directory to the `/root` of the docker container
COPY . /root 

# Can also copy individual files 
COPY app.py /root/
```

- Define the entrypoint or the command to run the application
```dockerfile
# If we want to serve file for example, challenge.txt 
CMD ["python", "-m", "http.server", "8080"]

# If the challenge is like a flask, django or guicorn server, run corresponding commands
CMD ["python", "app.py"]
# or 
CMD ["gunicorn", "myproject.wsgi:application", "--bind", "0.0.0.0:8000"]
```

## Example of Base64 challenge with Python script and Dockerfile

1. Save the python file as `gen_flag`
    ```python
    #!/usr/bin/env python
    import base64
    import sys 

    flag = sys.argv[1] # sys.argv[1] is the first cli argument
    flag_bytes = flag.encode()
    base64_bytes = base64.b64encode(flag_bytes)
    base64_string = base64_bytes.decode()

    # Use absolute path of file
    with open("/app/challenge.txt", "w") as f:
        f.write(base64_string)
    ```

1. Dockerfile
    ```dockerfile
    FROM python:slim

    # Copy all challenge relate files to /root
    WORKDIR /root 
    COPY . /root 

    # Make gen_flag executable
    RUN chmod +x /root/gen_flag

    # Copy all files which user can access to /app 
    WORKDIR /app

    EXPOSE 8080
    CMD ["python", "-m", "http.server", "8080"]
    ```

1. Build the image 
    ```bash
    docker build -t mybase64 .
    ```

1. Run the image
    ```bash 
    # We mapped 1234 port to 8080 port of the container
    docker run -d -p 1234:8080 --name myFirstCTF mybase64
    ```

1. Make the challenge dynamically and set the flag 
    ```bash 
    docker exec myFirstCTF /root/gen_flag flag_{myDynamicFlag}
    ```

1. Test the challenge (for this challenge open browser and visit localhost:1234)

1. If again want to change the flag, repeat step 5

1. Stop the container
    ```bash 
    docker stop myFirstCTF
    ```

1. Remove the container
    ```bash 
    docker rm myFirstCTF
    ```

## After making challenge
- Upload the challenge files to your GitHub repository
- Contact `Meetesh`, `Kartikey`, `Animesh` or `Abhishek` for the review
- Schedule a meet with us and get your challenge in the event 

## Rules 
- The scripting file name must be `gen_flag`
- Review is mandatory for having your challenge accepted in the event

## Details
|||
|-|-|
| Deadline | 7 January, 2024 |
| Target | 3-5 Challenges per person |
