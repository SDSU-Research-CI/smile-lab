# VS Code
This recipe schedules an interactive VS Code session on TIDE.
This is useful when you need to run multiple instances of VS Code or when estimating resource allocations (i.e. CPUs, GPUs, and memory) while developing and testing your code.
A pod is allowed to run for 6 hours, so if you need to run for longer than that consider using a [deployment or job](https://csu-tide.github.io/batch-jobs/#deployments).

A quick note on the choice of storage for this recipe.
For the home directory, we are using Ceph Block storage mounted at the path `/home/jovyan` which is ReadWriteOnce, meaning that you can only attach your home directory to one job at a time.
We will also be using File Storage mounted at the path `/data` which is ReadWriteMany, meaning that multiple jobs can access the data at the same time.
For more information on storage types, please see the [Storage Services Overview](https://csu-tide.github.io/storage-services/).

## Ingredients
- [Local installation of kubectl](../README.md#install-kubectl)
- Preferred container image (optional)

## Prep
1. Set your namespace in an environment variable `ns`:
    - macOS & Linux:
        - `ns=[your-namespace-here]`
    - Windows (PowerShell)
        - `$ns="[your-namespace-here]"`
    - *Note*: Make sure to remove the brackets '[' & ']'
1. (Optional) Browse the [available container images](https://csu-tide.github.io/jupyterhub/images) to see what software you want to run
1. (Optional) Edit the file `vscode-pod.yaml` and replace the value for the `image:` line with the url of your chosen container image

## Instructions

### One-time Steps
The steps in this section should only be followed if you do not already have a home directory in your namespace.
You will typically follow these steps if this is your first time.

1. Modify the file `template-pvc.yaml` and prepend your SDSUid (SDSU email) prefix on line 4, in front of "-home"
    - *I.E.*: If your SDSUid is maztec@sdsu.edu, line 4 should be modified to "maztec-home" 
1. Create your home directory:
    ```bash
    kubectl -n $ns apply -f template-pvc.yaml
    ```
    - *Note*: If you get a validation warning message like 'The PersistentVolumeClaim "-home" is invalid...', then please verify that you saved your changes to the file

### Running VS Code
1. Modify the file `vscode-pod.yaml` to attach your home directory in the spec.volumes.home entry (line 58) such that the value matches "[your-sdsuid-prefix]-home"
    - I.E.: "maztec-home"
1. (Optional) If you are running this in a shared namespace with other users, then we highly recommend that you append your SDSUid prefix to the metadata.name value (line 4)
    - *I.E*: If your SDSUid is maztec@sdsu.edu, line 4 should be modified to "vscode-gpu-maztec"
1. Schedule your VS Code job onto TIDE:
    ```bash
    kubectl -n $ns apply -f vscode-pod.yaml
    ```
    - *Note*: If you get a validation warning message like 'The Pod "vscode-gpu" is invalid:...', then please verify that you saved your changes to the file
1. Check the status of your pod:
    ```bash
    kubectl -n $ns get pods
    ```    
    - or
    ```bash
    kubectl -n $ns get pods --watch
    ```
    - *Note*: The `--watch` option must be ended with `ctrl + c`
    - You should see the following:
        ```
        NAME               READY   STATUS    RESTARTS   AGE
        vscode-gpu           1/1     Running   0          8m50s
        ```
    - *Note:* If you modified the pod name in step 1 above, then you should see that value instead
1. Forward port 8080 from the job running on TIDE to your local machine
    - *Note:* If you modified the pod name in step 1 above, then you must use that value for the pod name in the command below; I.E.: vscode-gpu-maztec
    ```bash
    kubectl -n $ns port-forward vscode-gpu 8080
    ```
    - *Note*: This command must be running the entire time that you wish to access VS Code, if you see a connection error message in your browser, check the status of the port-forward command
    - *Note*: VS Code will continue running regardless of running port-forward, meaning that if you disconnect, then you can reconnect at a later time (assuming the job has not been stopped nor reached its maximum runtime)
1. Open your web browser and navigate to [http://127.0.0.1:8080/?folder=/data](http://127.0.0.1:8080/?folder=/data)
1. Start using VS Code in the browser
    - *Note*: Assuming that you modified the job to attach your home directory, then any VS Code extensions you install should persist between sessions and when rescheduling a VS Code job

## Clean Up
1. Hit `ctrl + c` to stop your port-forward command
1. Stop your VS Code job: 
    ```bash
    kubectl -n $ns delete -f vscode-pod.yaml
    ```

## VS Code Deployment
Running VS Code in a deployment will allow it to run for up to two weeks.

Similarly to customizing the pod, you will first need to customize the deployment in the file `vscode-deployment.yaml`:
1. If you are running in a shared namespace, then you need to suffix the value of "vscode-gpu" with your SDSUid prefix on lines 4, 9 and 13.
    - *I.E*: If your SDSUid is maztec@sdsu.edu, lines 4, 9 and 13 should be modified to "vscode-gpu-maztec"
    - *Note*: These values must match, especially if multiple users are running deployments of VS Code simultaneously in order to avoid conflicts
1. Attach your home directory in the spec.template.spec.volumes.home entry (line 67) such that the value matches "[your-sdsuid-prefix]-home"
    - I.E. If your SDSUid is maztec@sdsu.edu, the value should be modified to "maztec-home"
    - *Note*: Your home directory may only be attached to one job at a time, so if your deployment does not schedule, then make sure your home directory is not in use by another job

You will then follow the same steps in the "Running VS Code" section above, just swapping out the filename `vscode-pod.yaml` for the deployment filename `vscode-deployment.yaml`.
    - *Note*: Deployments will add additional unique identifiers to their pod's name, so the pod name will look something like `vscode-gpu-[sdsuid]-7f6b645c84-h9v64`

## Rescheduling VS Code
You may reschedule the same job once it has been completed or reached its maximum runtime.

1. Ensure to clean up the previous job by deleting it:
    ```bash
    kubectl -n $ns delete -f vscode-pod.yaml
    ```
1. You can then schedule the same job again by following the directions in the section "Running VS Code" starting at step 3.

The same commands above would also apply to a deployment, just swap out the filename for `vscode-deployment.yaml`.
