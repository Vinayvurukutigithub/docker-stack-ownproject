pipeline {
  agent any
  environment {
    // Use your Docker Hub username here (you used "vinayvinnu24" earlier)
    REGISTRY   = "docker.io/vinayvinnu24"
    IMAGE_NAME = "mywebsite"
    STACK      = "website"
  }
  stages {
    stage('Build') {
      steps {
        sh '''
          # Replace marker in HTML so you can see which build is live
          sed -i "s|__BUILD__|$BUILD_NUMBER|g" index.html

          docker build -t $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER -t $REGISTRY/$IMAGE_NAME:latest .
        '''
      }
    }
    stage('Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo $PASS | docker login -u $USER --password-stdin
            docker push $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
            docker push $REGISTRY/$IMAGE_NAME:latest
          '''
        }
      }
    }
    stage('Deploy to Swarm') {
      steps {
        sh '''
          # Inject the exact image tag into the stack file
          cp stack.yml stack.gen.yml
          sed -i "s|IMAGE_PLACEHOLDER|$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER|g" stack.gen.yml

          # Deploy/Update the stack
          docker stack deploy -c stack.gen.yml $STACK
        '''
      }
    }
  }
  post {
    success {
      echo "Deployed $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER to stack $STACK"
    }
  }
}
