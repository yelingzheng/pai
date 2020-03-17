# Advanced Job Settings

## Parameters and Secrets

It is common to train models with different parameters. OpenPAI supports parameter definition and reference, which provides a flexible way of training and comparing models. You can define your parameters in the Parameters section and reference them by using <% $parameters.paramKey %> in your commands. For example, the following picture shows how to define the [Quick Start](/manual/cluster-user/advanced-jobs.md) job using a `stepNum` parameter.

   <img src="/manual/cluster-user/imgs/use-parameters.png" width="100%" height="100%" />

You can define batch size, learning rate, or whatever you want as parameters to accelerate your job submission.

In some cases, it is desired to define some secret messages such as password, token, etc. You can use the `Secrets` section for the information. The usage is the same as parameters except that secrets will not be displayed or recorded.

## Multiple Task Roles

If you use the `Distributed` button to submit jobs, the job can be defined as a multi-taskrole job.

   <img src="/manual/cluster-user/imgs/distributed-job.png" width="60%" height="60%" />

For single server jobs, there is only one role.

For distributed jobs, there may be multiple task roles. For example, when TensorFlow is used to running distributed job, it has two roles, including parameter server and worker. There are two task roles in the corresponding job configuration.

Instances is the number of instances of this task role. In distributed jobs, it depends on how many instances are needed for a task role. For example, if it's 8 in a worker role of TensorFlow. It means there should be 8 Docker containers for the worker role.

The picture below shows how to define task roles and instance numbers in a distributed job.

   <img src="/manual/cluster-user/imgs/taskrole-and-instance.png" width="100%" height="100%" />

### Environmental Variables and Port Reservation

In a distributed, one task might communicate with others. So a task need to be aware of other tasks' runtime information such as IP, port, etc. The system exposes such runtime information as environment variables to each task's Docker container. For mutual communication, user can write code in the container to access those runtime environment variables.

Below we show a complete list of environment variables accessible in a Docker container:

| Category          | Environment Variable Name                             | Description                                                                        |
| :---------------- | :---------------------------------------------------- | :--------------------------------------------------------------------------------- |
| Job level         | PAI_JOB_NAME                                          | `jobName` in config file                                                           |
|                   | PAI_USER_NAME                                         | User who submit the job                                                            |
|                   | PAI_DEFAULT_FS_URI                                    | Default file system uri in PAI                                                     |
| Task role level   | PAI_TASK_ROLE_COUNT                                   | Total task roles' number in config file                                            |
|                   | PAI_TASK_ROLE_LIST                                    | Comma separated all task role names in config file                                 |
|                   | PAI_TASK_ROLE_TASK_COUNT\_`$taskRole`                 | Task count of the task role                                                        |
|                   | PAI_HOST_IP\_`$taskRole`\_`$taskIndex`                | The host IP for `taskIndex` task in `taskRole`                                     |
|                   | PAI_PORT_LIST\_`$taskRole`\_`$taskIndex`\_`$portType` | The `$portType` port list for `taskIndex` task in `taskRole`                       |
|                   | PAI_RESOURCE\_`$taskRole`                             | Resource requirement for the task role in "gpuNumber,cpuNumber,memMB,shmMB" format |
|                   | PAI_MIN_FAILED_TASK_COUNT\_`$taskRole`                | `taskRole.minFailedTaskCount` of the task role                                     |
|                   | PAI_MIN_SUCCEEDED_TASK_COUNT\_`$taskRole`             | `taskRole.minSucceededTaskCount` of the task role                                  |
| Current task role | PAI_CURRENT_TASK_ROLE_NAME                            | `taskRole.name` of current task role                                               |
| Current task      | PAI_CURRENT_TASK_ROLE_CURRENT_TASK_INDEX              | Index of current task in current task role, starting from 0                        |

Some environmental variables are in association with ports. In OpenPAI, you can reserve ports for each container in advanced settings, as shown in the image below:

   <img src="/manual/cluster-user/imgs/advanced-and-port.png" width="100%" height="100%" />

The ports you reserved are available in environmental variables like `PAI_PORT_LIST_$taskRole_$taskIndex_$portLabel`, where `$taskIndex` means the instance index of that task role.

## Job Exit Spec, Retry Policy, and Completion Policy

There are always different kinds of errors in jobs. In OpenPAI, errors are classified into 3 categories automatically:

  1. Transient Error: This kind of error is considered transient. There is high chance to bypass it through a retry. 
  2. Permanent Error: This kind of error is considered permanent. Retries may not help.
  3. Unknown Error: Errors besides transient error and permanent error.

In jobs, transient error will be always retried, and permanent error will never be retried. If unknown error happens, PAI will retry it according to user settings. To set a retry policy and completion policy for your job, please toggle the `Advanced` mode, as shown in the following image:

   <img src="/manual/cluster-user/imgs/advanced-and-retry.png" width="100%" height="100%" />

Here we have 3 settings: `Retry count`, `Task retry count`, and `Completion policy`. To date, we should be aware that a job is made up by multiple tasks. One task stands for a single instance in a task role. `Task retry count` is used for task-level retry. `Retry count` and `Completion policy` are used for job-level retry.

Firstly, let's look at `Retry count` and `Completion policy`. 

In `Completion policy`, there is settings for `Min Failed Instances` and `Min Succeed Instances`. `Min Failed Instances` means number of failed tasks to fail the entire job. It should be -1 or no less than 1. If it is set to -1, the job will always succeed regardless any task failure. Default value is 1, which means 1 failed task will cause an entire failure. `Min Succeed Instances` means number of succeeded tasks to succeed the entire job. It should be -1 or no less than 1. If it is set to -1, the job will only succeed until all tasks are completed and minFailedInstances is not triggered. Default value is -1. 

If a job doesn't succeed after the check of `Completion policy`, the failure is caused by an unknown error, and `Retry count` is larger than 0, the whole job will be retried. Set `Retry count` to a larger number if you need more retries.

Finally, for `Task retry count`, it is the desired retry number for a single task. A special notice is that, this setting won't work unless you set `extras.gangAllocation` to `false` in the [job protocol](#Job-Protocol-Export-and-Import-Jobs).

## Job Protocol, Export and Import Jobs

In OpenPAI, all jobs are represented by [YAML](https://yaml.org/), a markup language. You can click the button Edit YAML below to edit the YAML definition directly. You can also export and import YAML files using the `Export` and `Import` button.

   <img src="/manual/cluster-user/imgs/export-and-import.png" width="100%" height="100%" />

To see a full reference of job protocol, please check [openpai-protocol](https://github.com/microsoft/openpai-protocol/blob/master/schemas/v2/schema.yaml).

## Distributed Job Examples

### TensorFlow CIFAR10

This example is a TensorFlow CIFAR-10 training job, which runs a parameter server and a worker. This job needs at least 5 GPUs to run. Please refer to [tensorflow-cifar10.yaml](https://github.com/microsoft/pai/blob/master/marketplace-v2/tensorflow-cifar10.yaml).

### Horovod PyTorch

This example, [horovod-pytorch-synthetic-benchmark.yaml](https://github.com/microsoft/pai/blob/master/marketplace-v2/horovod-pytorch-synthetic-benchmark.yaml), is a Horovod benchmark using PyTorch and Open MPI. Please make sure the IFNAME settings fit your environment. It needs at least 8 GPUs to run this job. 

## RDMA Jobs

## InfiniBand Jobs

## Reference

 - [Job Protocol](https://github.com/microsoft/openpai-protocol/blob/master/schemas/v2/schema.yaml)
 - [PAI Job Exit Spec User Manual](https://github.com/microsoft/pai/blob/master/src/k8s-job-exit-spec/config/user-manual.md)
 - [Retry Policy](https://github.com/microsoft/frameworkcontroller/blob/master/doc/user-manual.md#retrypolicy)