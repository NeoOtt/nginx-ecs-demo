stage('Deploy') {
    steps {
        script {
            withAWS(credentials: 'aws-lab', region: params.AWS_REGION) {
                // Obtener la definición actual
                def taskDef = sh(
                    script: "aws ecs describe-task-definition --task-definition ${params.TASK_FAMILY} --region ${params.AWS_REGION} --output json",
                    returnStdout: true
                ).trim()

                // Extraer solo los campos necesarios para register-task-definition
                def filteredDef = sh(
                    script: "echo '${taskDef}' | jq '.taskDefinition | {family, taskRoleArn, executionRoleArn, networkMode, containerDefinitions, volumes, placementConstraints, requiresCompatibilities, cpu, memory, pidMode, ipcMode, proxyConfiguration, inferenceAccelerators, ephemeralStorage, runtimePlatform, enableFaultInjection}'",
                    returnStdout: true
                ).trim()

                // Actualizar la imagen del contenedor
                def updatedDef = sh(
                    script: "echo '${filteredDef}' | jq '.containerDefinitions[0].image = \"${ECR_URL}/${params.ECR_REPO}:${IMAGE_TAG}\"'",
                    returnStdout: true
                ).trim()

                // Guardar y registrar
                sh "echo '${updatedDef}' > task-def.json"
                def newTaskDef = sh(
                    script: "aws ecs register-task-definition --region ${params.AWS_REGION} --cli-input-json file://task-def.json --output json",
                    returnStdout: true
                ).trim()

                def newArn = sh(
                    script: "echo '${newTaskDef}' | jq -r '.taskDefinition.taskDefinitionArn'",
                    returnStdout: true
                ).trim()

                sh "aws ecs update-service --cluster ${params.ECS_CLUSTER} --service ${params.ECS_SERVICE} --task-definition ${newArn} --region ${params.AWS_REGION}"
            }
        }
    }
}