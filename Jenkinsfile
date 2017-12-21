def gitCommit() {
        sh "git rev-parse HEAD > GIT_COMMIT"
        def gitCommit = readFile('GIT_COMMIT').trim()
        sh "rm -f GIT_COMMIT"
        return gitCommit
    }

    node {
        // Checkout source code from Git
        stage 'Checkout'
        checkout scm

        // Build Docker image
        stage 'Build'
        sh "docker build -t dtr.novalocal/root/gitlab-demo:${gitCommit()} ."

        // Login to DTR 
        stage 'Login'
        withCredentials(
            [[
                $class: 'UsernamePasswordMultiBinding',
                credentialsId: 'dtr',
                passwordVariable: 'DTR_PASSWORD',
                usernameVariable: 'DTR_USERNAME'
            ]]
        ){ 
        sh "docker login -u ${env.DTR_USERNAME} -p ${env.DTR_PASSWORD}  dtr.novalocal"}

        // Push the image 
        stage 'Push'
        sh "docker push dtr.novalocal/root/gitlab-demo:${gitCommit()}"

        // run the container
        stage 'Deploy'
        sh "export DOCKER_HOST=tcp://master1.novalocal:443 && export DOCKER_CERT_PATH=/client && export DOCKER_TLS_VERIFY=1 && docker service create --name nginx --network new-hrm-network --publish target=80,published=8005 --label com.docker.ucp.mesh.http=external_route=http://nginx.example.org,internal_port=80 --constraint 'node.role == worker' dtr.novalocal/root/gitlab-demo:${gitCommit()}"
    }

