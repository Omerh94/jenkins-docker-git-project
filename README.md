# jenkins-docker-git-projectpoll test
poll test 2
poll-check
	This project creates a Docker image from Dockerfile
		Jenkins pipeline builds and runs container
	  Container sleeps for 10 seconds
		After 20 seconds Jenkins checks container state
	  If exited with 0 → success
	  If still running → kill and fail
	  Triggered automatically by SCM change
