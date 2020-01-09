def remote = [:]
remote.name = "api-server"
remote.host = "ec2-52-215-220-174.eu-west-1.compute.amazonaws.com"
remote.allowAnyHosts = true
node {
    withCredentials([
	sshUserPrivateKey(credentialsId: 'cle-ssh-de-l-api', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName'),
	usernamePassword(credentialsId: 'gitApi', usernameVariable: 'USERGIT',passwordVariable: 'PASSGIT')
	]){
        remote.user = userName
        remote.identityFile = identity			
				
        stage("Deploy dev env step") {
		sshCommand remote: remote, command: 'echo "##################" Backup #################"'
		sshCommand remote: remote, sudo: true, command: 'cp -R app/api-my-pet app/api-my-pet-old'

            	sshCommand remote: remote, command: 'echo "##################" Start Git pull #################"'
		sh 'echo git pull https://${USERGIT}:${PASSGIT}@bitbucket.org/founa44/api-my-pet.git > gitpull.sh'
		sshPut remote: remote, from: 'gitpull.sh', into: '/home/ec2-user/app/api-my-pet'
		writeFile file: 'test.sh', text: 'test'
		sshPut remote: remote, from: 'test.sh', into: '/home/ec2-user/app'
            	sshCommand remote: remote, command: 'cd app/api-my-pet && cat /home/ec2-user/app/api-my-pet/gitpull.sh | bash'
            	sshCommand remote: remote, command: 'cd app/api-my-pet && ls -al'
		sshRemove remote: remote, path: '/home/ec2-user/app/api-my-pet/gitpull.sh'

            	sshCommand remote: remote, command: 'echo "##################" Start Build #################"'
		sshCommand remote: remote, command: 'dotnet --info'
		sshCommand remote: remote, sudo: true, command: 'dotnet build /home/ec2-user/app/api-my-pet/MyPetAPI'
		sshCommand remote: remote, sudo: true, command: 'dotnet publish /home/ec2-user/app/api-my-pet/MyPetAPI'
						
		sshCommand remote: remote, command: 'echo "##################" Start Deploy #################"'
		sshCommand remote: remote, sudo: true, command: 'cp -a app/api-my-pet/MyPetAPI/bin/Debug/netcoreapp3.0/publish/ /var/api-my-pet/publish'
		sshCommand remote: remote, sudo: true, command: 'chmod -R 755 /var/api-my-pet/publish'
		sshCommand remote: remote, sudo: true, command: 'systemctl daemon-reload'
		sshCommand remote: remote, sudo: true, command: 'systemctl restart kestrel-mypet.service'
		sshCommand remote: remote, sudo: true, command: 'service httpd restart'
	
            	sshCommand remote: remote, command: 'echo "##################" END Build + deploy #################"'
        }
    }
}
