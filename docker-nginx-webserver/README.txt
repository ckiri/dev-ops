# Build the docker image
docker build -t nginx:webserver .

# Start nginx webserver with following command:
docker run -p 80:80 -v "$(pwd)"/index.html:/var/www/html/index.html --name webserver -d nginx:webserver
