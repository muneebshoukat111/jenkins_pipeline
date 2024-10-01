pipeline:
  agent:
    docker:
      image: "docker:latest"
      args: "--privileged"  # Needed for Docker-in-Docker if applicable
  stages:
    - stage: "Build Docker Image"
      steps:
        - script: |
            # Build the Docker image
            docker build -t your-image-name:latest .
