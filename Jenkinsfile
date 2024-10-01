pipeline:
  agent:
    docker:
      image: "docker:latest"
      args: "--privileged" 
  stages:
    - stage: "Build Docker Image"
      steps:
        - script: |
            # Build the Docker image
            docker build -t your-image-name:latest .
