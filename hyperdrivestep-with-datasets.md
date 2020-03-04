# Using Datasets with HyperDriveStep

This example does only show relevant code snippets, see [this notebook](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/machine-learning-pipelines/intro-to-pipelines/aml-pipelines-parameter-tuning-with-hyperdrive.ipynb) for the full example.

First, reference data using `Dataset` class:

```python
from azureml.core import Dataset
dataset = Dataset.get_by_name(ws, name='mnist')
```

Setup `TensorFlow` estimator. Make sure to include `azureml-dataprep[fuse,pandas]`, which is needed to mount `Dataset` to the AML Compute Cluster:  

```python
est = TensorFlow(source_directory=script_folder,                 
                 compute_target=compute_target,
                 entry_script='tf_mnist.py', 
                 use_gpu=True,
                 pip_packages=['azureml-dataprep[fuse,pandas]'],
                 framework_version='1.13')
```

Pass in `Dataset` as a mount:

```python
hd_step = HyperDriveStep(
    name=hd_step_name,
    hyperdrive_config=hd_config,
    estimator_entry_script_arguments=['--data-folder', dataset.as_named_input("mnist_data").as_mount()],
    inputs=[],
    metrics_output=metrics_data)
```

This will mount the dataset during startup to the AML Compute Cluster:

```
Initialize DatasetContextManager.
Starting the daemon thread to refresh tokens in background for process with pid = 145
Set Dataset mnist_data's target path to /tmp/tmpxxea0sse
Enter __enter__ of DatasetContextManager
Processing 'mnist_data'
Processing dataset FileDataset
{
  "source": [
    "('workspaceblobstore', 'mnist/')"
  ],
  "definition": [
    "GetDatastoreFiles"
  ],
  "registration": {
    "id": "498361e4-f1c4-419c-b314-f0006424ed7e",
    "name": "mnist",
    "version": 1,
    "workspace": "Workspace.create(name='aml-demo', subscription_id='43ab27bb-ee6c-4f68-b9cf-a26c4c454a4a', resource_group='aml-demo')"
  }
}
Mounting mnist_data to /tmp/tmpxxea0sse
Mounted mnist_data to /tmp/tmpxxea0sse
```
