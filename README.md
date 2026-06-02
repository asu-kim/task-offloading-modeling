# Modeling AI inference tasks scheduling problem
## Functionalities
This repo provides a visualization tool that enables users to test scheduling various tasks on heterogeneous devices.

## Requirements
```
sudo apt install python3-tk
```
```
pip install pandas
```

## TODOs
 - [ ] Build a system that shows the simulation results of the given number of devices, tasks, and task configurations.

## Simulation
Use logical delay to emulate communication cost

## Deployment

### A6000
```
./my_env/bin/python3.10 -m pip install --index-url https://download.pytorch.org/whl/cu128 torch torchvision
```


```
export PYTHONPATH="$PWD/my_env/lib/python3.10/site-packages:$PYTHONPATH"
```