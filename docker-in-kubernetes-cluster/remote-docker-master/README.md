# build & publish docker images on mach1

## Prerequisites
On hpclogin1.hpc.ford.com:
1. Access to `kubectl` command. Execute `kubectl login`. The password should be the same as your HPC
login password.

2. `/s/$USER/.docker/config.json` has a field for the desired docker registries.  Example:
    ```
    {
            "auths": {
                    "hpcregistry.hpc.ford.com": {
                            "auth": "foo"
                    },
                    "registry.ford.com": {
                            "auth": "bar"
                    }
     ...
    ```
    
    If you do not have access to a `.docker/config.json`, use the alternative option in step 1 of setup below.

    <details>
    
      <summary>Docker Registries </summary>
      
    `hpcregistry.hpc.ford.com` corresponds to the Portus registry. `registry.ford.com` corresponds
    to the Quay registry. If you need to push/pull images from a certain registry and it is not
    present in `/s/$USER/.docker/config.json`, then execute `docker login hpcregistry.hpc.ford.com`
    for Portus. For Quay, you can find the docker login command by navigating to the robot account 
    under `Settings` and clicking on the `Docker Login` tab.
    
    </details>
    
3. `/s/$USER/.ssh/id_rsa.pub` and `/s/$USER/.ssh/id_rsa` exists, has access to the desired git repositories, and does not have a passphrase.

    <details>
    
      <summary>Generating SSH Keys </summary>
      
    If your SSH key has a passphrase, you can generate a new one without a passphrase following the
    example [here](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
    
    After generating a new SSH key, make sure to update the key in your GitHub account and copy the
    `id_rsa` and `id_rsa.pub` files to `/s/$USER/.ssh`.
    
    </details>
    
## Setup
On hpclogin1.hpc.ford.com:
1. Create the following kubernetes secrets

    ```bash
    #docker regisry credentials to allow kubernetes to pull images
    kubectl create secret generic docker-registry-credentials \
        --from-file=.dockerconfigjson=$(readlink -f /s/$USER/.docker/config.json) \
        --type=kubernetes.io/dockerconfigjson

    #docker registry credentials inside container
    kubectl create secret generic docker-registry-credentials-yaml \
        --from-file=config.json=$(readlink -f /s/$USER/.docker/config.json)

    #ssh keys
    kubectl create secret generic ssh-keys \
        --from-file=id_rsa=$(readlink -f /s/$USER/.ssh/id_rsa) \
        --from-file=id_rsa.pub=$(readlink -f /s/$USER/.ssh/id_rsa.pub)
    ```
    
    <details>
    
      <summary>Alternative (explicit passwords)</summary>
      
    ```bash
    #docker regisry credentials to allow kubernetes to pull images
    kubectl create secret docker-registry docker-registry-credentials --docker-server=hpcregistry.hpc.ford.com --docker-username=your_cdsid --docker-password=your_hpc_password

    #docker registry credentials inside container
    #note that this method only support access to one registry from within the container
    kubectl create secret docker-registry docker-registry-credentials-yaml --docker-server=your_docker_registry --docker-username=your_cdsid --docker-password=your_password

    #ssh keys
    kubectl create secret generic ssh-keys \
        --from-file=id_rsa=$(readlink -f /s/$USER/.ssh/id_rsa) \
        --from-file=id_rsa.pub=$(readlink -f /s/$USER/.ssh/id_rsa.pub)
    ```
    
    </details>

2. `git clone git@github.ford.com:JPOSTEL1/remote-docker.git`.

    <details>
    
      <summary>Git Clone in HPC Environment </summary>
      
    If you cannot clone git repositories in the HPC environment, then you might have an issue with
    your SSH key for GitHub.
    
    If your SSH key has a passphrase, you can generate a new one without a passphrase following the
    example [here](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
    
    After generating a new SSH key, make sure to update the key in your GitHub account and copy the
    `id_rsa` and `id_rsa.pub` files to `/s/$USER/.ssh`. If you are still not able to git clone 
    repos, try adding the `id_rsa` and `id_rsa.pub` files to `/u/$USER/.ssh` as well.
    
    </details>
    
3. `kubectl apply -f remote-docker/remote-docker.yml`

4. `kubectl exec -it remote-docker-xxxx bash`
    * get xxxx by running `kubectl get pods`
    * this command will not work until the pod is initialized 

5. Use `git`, `vim`, and `docker` to your heart's content
