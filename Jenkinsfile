pipeline {
  agent any
triggers { pollSCM('H/1 * * * *') }
  environment {
    IMAGE_NAME     = "sleep-test-image"
    CONTAINER_NAME = "sleep-test-${BUILD_NUMBER}"   // custom name via env
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
      }
    }

    stage('Run Container (check name not exists)') {
      steps {
        sh '''
          set -e

          # If a container with the same name exists (running or stopped) -> remove it
          if docker ps -a --format '{{.Names}}' | grep -x "${CONTAINER_NAME}" >/dev/null 2>&1; then
            echo "Container ${CONTAINER_NAME} already exists. Removing..."
            docker rm -f "${CONTAINER_NAME}" || true
          fi

          # Run container in background with custom name
          docker run -d --name "${CONTAINER_NAME}" "${IMAGE_NAME}:${BUILD_NUMBER}"
        '''
      }
    }

    stage('Wait 20s and Check Result') {
      steps {
        sh '''
          set -e
          echo "Waiting 20 seconds..."
          sleep 20

          # Check if still running
          RUNNING=$(docker inspect -f '{{.State.Running}}' "${CONTAINER_NAME}" 2>/dev/null || echo "false")

          if [ "$RUNNING" = "true" ]; then
            echo "test failed: container is still running -> killing ${CONTAINER_NAME}"
            docker kill "${CONTAINER_NAME}" || true
            exit 1
          fi

          # Container finished, check exit code
          EXIT_CODE=$(docker inspect -f '{{.State.ExitCode}}' "${CONTAINER_NAME}" 2>/dev/null || echo "1")

          if [ "$EXIT_CODE" = "0" ]; then
            echo "test run success: container finished with exit code 0"
            exit 0
          else
            echo "test failed: container finished with exit code ${EXIT_CODE}"
            exit 1
          fi
        '''
      }
    }
  }

  post {
    always {
      sh 'docker rm -f "${CONTAINER_NAME}" >/dev/null 2>&1 || true'
    }
  }
}
