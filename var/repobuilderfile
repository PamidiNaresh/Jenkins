

  parameters {
    booleanParam(defaultValue: true, description: 'Uncheck if you do not want to make a Bitbucket repository', name: 'Create Repository')
    string(name: 'Application Name', description: 'Name of the application, max character length is TBD.')
    choice(name: 'Project Name', description: 'Please pick one', choices: ['AC','AB','BRE', 'BRED', 'BREF', 'BREP', 'BRESI', 'BRECM', 'CR', 'DATA', 'EVD', 'IOT', 'MENU', 'QE', 'SVCS', 'SRE', 'TEST', 'TFM', 'TF', 'TL'])
    choice(name: 'Build Type', choices: ['nobuild', 'ansible', 'docker', 'helm', 'lambda', 'terraform', 'terraform-module', 'npm-module', 'OpenTestSelenium', 'OpenTestUI', 'jarmenu']) //New build-type added
    choice(name: 'projectsview', description: 'Please pick one', choices: ['AC Projects','AB Projects','BRE Projects', 'BRED Projects', 'BREF Projects', 'BREP Projects', 'BRESI Projects', 'BRECM Projects', 'CR Projects', 'DATA Projects', 'EVD Projects', 'IOT Projects', 'MENU Projects', 'QE Projects', 'SVCS Projects', 'SRE Projects', 'TEST Projects', 'TFM Projects', 'TF Projects', 'TL Projects'])
    string(name: 'githubid', description: 'Id will be unique identifier ex 34343434')
  }

  stages {
    stage("Pre-check") {
      steps {
        script {
          try {
            if (params['Application Name'] == "") {
              error("Build failed because Application Name is missing")
            } else if (params['Application Name'] =~ "${regexValidator}") {
              error("Application name consists of special characters that are not allowed")
            }
            projectsFolder = "${params['Project Name']} Projects"
            destProject = "${params['Project Name']} ${params['Application Name']} ${params['Build Type']}".replaceAll("[\\s+\\_]", "-").toLowerCase()
            destGit  = "${params['Project Name']}/${destProject}"
            switch (params['Build Type'].toLowerCase()) {
              case 'docker':
                srcGit = "bre/bre-sample-docker"
                break
              case 'helm':
                srcGit = "bre/bre-sample-helm"
                break
              case 'terraform':
                srcGit = "bre/bre-sample-terraform"
                break
              case 'terraform-module':
                srcGit = "bre/bre-sample-terraform"
                break
              case 'lambda':
                srcGit = "bre/bre-sample-terraform"
                break
              case 'ansible':
                srcGit = "bre/bre-sample-ansible"
                break
              case 'nobuild':
                srcGit = "bre/bre-sample-nobuild"
                break
              case 'npm-module':
                srcGit = "bre/bre-sample-npm-module"
                break
              case 'opentestselenium':
                srcGit = "bre/bre-sample-OpenTestSelenium"
                break
              case 'jarmenu':
                srcGit = "bre/bre-sample-jarmenu"
                break
              case 'opentestui':
                srcGit = "bre/bre-sample-opentestUI"
            }
          } catch (Exception e) {
            ansiColor('xterm') {
                sh """
                set +x
                RED='\033[0;31m'
                BOLD='\033[1m'
                NC='\033[0m'
                echo -e "\${RED}+===================================[ERROR]===================================+\${NC}"
                echo -e "\${BOLD}The pipeline is missing one or more required parameters.\${NC}"
                echo -e "If this is the first run, please re-run this pipeline."
                echo -e "\${BOLD}Error Message:\${NC}"
                echo -e "${e.getMessage()}"
                echo -e "\${RED}+=============================================================================+\${NC}"
                """
            }
            error("Build failed because of an error")
          }
        }
      }
    }
    //Create New Repo
    stage("Create new BitBucket repo") {
      when {
          //Only run if Create Repository is true
          expression { params['Create Repository'] == true }
      }
      steps {
        container ('build-tools') {
          script {
            repoBuilder.createNewRepo(params['Project Name'], destGit)
          }
        }
      }
    }

    //Clone repo into Jenkins workspace
    stage("Clone src repo") {
      when {
          //Only run if Create Repository is true
          expression { params['Create Repository'] == true }
      }
      steps {
        container ('build-tools') {
          script {
            repoBuilder.cloneSrcRepo(srcGit)
          }
        }
      }
    }

    //Sync cloned repo in Jenkins workspace to newly created Bitbucket repo
    stage ("Sync src to dest repo") {
      when {
          //Only run if Create Repository is true
          expression { params['Create Repository'] == true }
      }
      steps {
        container ('build-tools') {
          script {
            repoBuilder.syncSrcToDestRepo(srcGit, destGit)
          }
        }
      }
    }

    // Create the new pipeline with a default script
    stage("Create new job in folder") {
      steps {
        container('build-tools') {
          script {
            if (params['Build Type'].toLowerCase() == 'nobuild') {
              echo "INFO: Not creating jenkins job"
            } else {
              if (multiBranchPipeline) {
                repoBuilder.createNewJenkinsView(params['Project Name'])
                repoBuilder.createNewJenkinsFolder(projectsFolder, params['Project Name'])
                repoBuilder.createNewJenkinsJobWithMultiBranch(params['projectsView'], params['Project Name'], destProject, destGit, params['githubid'])
              } else {
                repoBuilder.createNewJenkinsView(params['Project Name'])
                repoBuilder.createNewJenkinsFolder(projectsFolder, params['Project Name'])
                repoBuilder.createNewJenkinsJob(projectsFolder, params['Project Name'], destProject, destGit)
              }
            }
          }
        }
      }
    }
  }
}
