build:
	sudo docker build -t webserver .

clean:
	sudo docker stop webserver &
	sudo docker rm -f webserver &
	sudo docker rmi -f webserver &

run: clean build
	sudo docker run -d -p 8080:8888 --name webserver webserver

	
	 
