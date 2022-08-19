---
title: "Vehicle Model Creation"
date: 2022-05-09T13:43:25+05:30
weight: 2
description: >
  Learn how to create a Vehicle Model to get to access vehicle data or execute remote procedure calls.
aliases:
  - /docs/tutorials/tutorial_how_to_create_a_vehicle_model.md
  - /docs/python-sdk/tutorial_how_to_create_a_vehicle_model.md
---

A _Vehicle Model_ makes it possible to easily get vehicle data from the KUKSA Data Broker and to execute remote procedure calls over gRPC against _Vehicle Services_ and other _Vehicle Apps_. It is generated from the underlying semantic models for a concrete programming language as a graph-based, strongly-typed, intellisense-enabled library. 

This tutorial will show you how to:

- Create a _Vehicle Model_
- Add a _Vehicle Service_ to the _Vehicle Model_
- Distribute your Python Vehicle Model

{{% alert title="Note" %}}
A _Vehicle Model_ should be defined in its own package. This makes it possible to distribute the _Vehicle Model_ later as a standalone package and to use it in different _Vehicle App_ projects.
{{% /alert %}}

## Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/) with the [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python) installed. For information on how to install extensions on Visual Studio Code, see [VS Code Extension Marketplace](https://code.visualstudio.com/docs/editor/extension-gallery).

## Create a Vehicle Model from VSS specification

A _Vehicle Model_ can be generated from a [COVESA Vehicle Signal Specification](https://covesa.github.io/vehicle_signal_specification/) (VSS). VSS introduces a domain taxonomy for vehicle signals, in the sense of classical attributes, sensors and actuators with the raw data communicated over vehicle buses and data. The Velocitas [vehicle-model-generator](https://github.com/eclipse-velocitas/vehicle-model-generator) creates a _Vehicle Model_ from the given specification and generates a package for use in _Vehicle App_ projects.

Follow the steps to generate a _Vehicle Model_.

  1. Clone the [vehicle-model-generator](https://github.com/eclipse-velocitas/vehicle-model-generator) repository in a container volume.

  2. In this container volume, clone the [vehicle-signal-specification](https://github.com/COVESA/vehicle_signal_specification) repository and if required checkout a particular branch: 

        ```bash
        git clone https://github.com/COVESA/vehicle_signal_specification

        cd vehicle_signal_specification
        git checkout <branch-name>
        ```
        In case the VSS vspec doesn't contain the required signals, you can create a vspec using the [VSS Rule Set](https://covesa.github.io/vehicle_signal_specification/rule_set/). 
        
  3. Execute the command

        ```bash
        python3 gen_vehicle_model.py -I ./vehicle_signal_specification/spec ./vehicle_signal_specification/spec/VehicleSignalSpecification.vspec -T sdv_model -N sdv_model
        ```

        This creates a `sdv_model` directory in the root of repository along with a `setup.py` file.
        To have a custom model name, refer to README of [vehicle-model-generator](https://github.com/eclipse-velocitas/vehicle-model-generator) repository.
  4. Change the version of package in `setup.py` manually (defaults to 0.1.0).
  5. Now the newly generated `sdv_model` can be used for distribution. (See [Distributing your Python Vehicle Model](#distributing-your-python-vehicle-model))

## Create a Python Vehicle Model Manually

Alternative to the generation from a VSS specification you could create the _Vehicle Model_ manually. The following sections describing the required steps.

### Setup a Python Package manually

  A _Vehicle Model_ should be defined in its own Python Package. This allows to distribute the _Vehicle Model_ later as a standalone package and to use it in different _Vehicle App_ projects.

  The name of the _Vehicle Model_ package will be `my_vehicle_model` for this walkthrough.

  1. Start Visual Studio Code
  2. Select **File > Open Folder (File > Open**... on macOS) from the main menu.
  3. In the Open Folder dialog, create a `my_vehicle_model` folder and select it. Then click Select Folder (Open on macOS).
  4. Create a new file `setup.py` under `my_vehicle_model`:

     ```python
     from setuptools import setup

     setup(name='my_vehicle_model',
         version='0.1',
         description='My Vehicle Model',
         packages=['my_vehicle_model'],
         zip_safe=False)
     ```

     This is the Python package distribution script.

  5. Create an empty folder `my_vehicle_model` under `my_vehicle_model`.
  6. Create a new file `__init__.py` under `my_vehicle_model/my_vehicle_model`.

  At this point the source tree of the Python package should look like this:

  ```
  my_vehicle_model
  ├── my_vehicle_model
  │   └── __init__.py
  └── setup.py
  ```

  To verify that the package is created correctly, install it locally:

  ```bash
  pip3 install .
  ```

  The output of the above command should look like this:

  ```
  Defaulting to user installation because normal site-packages is not writeable
  Processing /home/user/projects/my-vehicle-model
  Preparing metadata (setup.py) ... done
  Building wheels for collected packages: my-vehicle-model
  Building wheel for my-vehicle-model (setup.py) ... done
  Created wheel for my-vehicle-model: filename=my_vehicle_model-0.1-py3-none-any.whl size=1238 sha256=a619bc9fbea21d587f9f0b1c1c1134ca07e1d9d1fdc1a451da93d918723ce2a2
  Stored in directory: /home/user/.cache/pip/wheels/95/c8/a8/80545fb4ff73c974ac1716a7bff6f7f753f92022c41c2e376f
  Successfully built my-vehicle-model
  Installing collected packages: my-vehicle-model
  Successfully installed my-vehicle-model-0.1
  ```

  Finally, uninstall the package again:

  ```bash
  pip3 uninstall my_vehicle_model
  ```

### Add Vehicle Models manually

  1. Install the Python Vehicle App SDK:

          ```bash
          pip3 install git+https://github.com/eclipse-velocitas/vehicle-app-python-sdk.git@v0.4.0
          ```

          The output of the above command should end with:

          ```bash
          Successfully installed sdv-0.4.0
          ```

      Now it is time to add some _Vehicle Models_ to the Python package. At the end of this section you will have a _Vehicle Model_, that contains a `Cabin` model, a `Seat`model and has the following tree structure:

          Vehicle
          └── Cabin
              └── Seat (Row, Pos)

  2. Create a new file `Seat.py` under `my_vehicle_model/my_vehicle_model`:

      ```python
      from sdv.model import Model

      class Seat(Model):

          def __init__(self, parent):
              super().__init__(parent)
              self.Position = DataPointFloat("Position", self)

      ```

      This creates the Seat model with a single data point of type `float` named `Position`.

  3. Create a new file `Cabin.py` under `my_vehicle_model/my_vehicle_model`:

      ```python
      from sdv.model import Model

      class Cabin(Model):

          def __init__(self, parent):
              super().__init__(parent)

              self.Seat = ModelCollection[Seat](
                  [NamedRange("Row", 1, 2), NamedRange("Pos", 1, 3)], Seat(self)
          )
      ```

      This creates the `Cabin` model, which contains a set of six `Seat` models, referenced by their rows and positions:

      - row=1, pos=1
      - row=1, pos=2
      - row=1, pos=3
      - row=2, pos=1
      - row=2, pos=2
      - row=2, pos=3

  4. Create a new file `vehicle.py` under `my_vehicle_model/my_vehicle_model`:

      ```python
      from sdv.model import Model
      from my_vehicle_model.Cabin import Cabin


      class Vehicle(Model):
          """Vehicle model"""

          def __init__(self):
              super().__init__()
              self.Speed = DataPointFloat("Speed", self)
              self.Cabin = Cabin(self)

      vehicle = Vehicle()

      ```

  The root model of the _Vehicle Model_ tree should be called _Vehicle_ by convention and is specified, by setting parent to `None`. For all other models a parent model must be specified as the 2nd argument of the `Model` constructor, as can be seen by the `Cabin` and the `Seat` models above.

  A singleton instance of the _Vehicle Model_ called `vehicle` is created at the end of the file. This instance is supposed to be used in the _Vehicle Apps_. Creating multiple instances of the _Vehicle Model_ should be avoided for performance reasons.

## Add a Vehicle Service

_Vehicle Services_ provide service interfaces to control actuators or to trigger (complex) actions. E.g. they communicate with the vehicle internals networks like CAN or Ethernet, which are connected to actuators, electronic control units (ECUs) and other vehicle computers (VCs). They may provide a simulation mode to run without a network interface. _Vehicle Services_ may feed data to the Data Broker and may expose gRPC endpoints, which can be invoked by _Vehicle Apps_ over a _Vehicle Model_.

In this section, we add a _Vehicle Service_ to the _Vehicle Model_. 

1. Create a new folder `proto` under `my_vehicle_model/my_vehicle_model`.
2. Copy your proto file under `my_vehicle_model/my_vehicle_model/proto`

   As example you could use the protocol buffers message definition [seats.proto](https://github.com/eclipse/kuksa.val.services/blob/main/seat_service/proto/sdv/edge/comfort/seats/v1/seats.proto) provided by the KUKSA VAL services which describes a [seat control service](https://github.com/eclipse/kuksa.val.services/tree/main/seat_service).

3. Install the grpcio tools including mypy types to generate the python classes out of the proto-file:

   ```bash
   pip3 install grpcio-tools mypy_protobuf
   ```

4. Generate Python classes from the `SeatService` message definition:

   ```bash
   python3 -m grpc_tools.protoc -I my_vehicle_model/proto --grpc_python_out=./my_vehicle_model/proto --python_out=./my_vehicle_model/proto --mypy_out=./my_vehicle_model/proto my_vehicle_model/proto/seats.proto
   ```

   This creates the following grpc files under the `proto` folder:

   - seats_pb2.py
   - seats_pb2_grpc.py
   - seats_pb2.pyi

5. Create the `SeatService` class and wrap the gRPC service:

   ```python
   from sdv.model import Service

   from my_vehicle_model.proto.seats_pb2 import (
       CurrentPositionRequest,
       MoveComponentRequest,
       MoveRequest,
       Seat,
       SeatComponent,
       SeatLocation,
   )
   from my_vehicle_model.proto.seats_pb2_grpc import SeatsStub


   class SeatService(Service):
       "SeatService model"

       def __init__(self):
           super().__init__()
           self._stub = SeatsStub(self.channel)

       async def Move(self, seat: Seat):
           response = await self._stub.Move(MoveRequest(seat=seat), metadata=self.metadata)
           return response

       async def MoveComponent(
           self,
           seatLocation: SeatLocation,
           component: SeatComponent,
           position: int,
       ):
           response = await self._stub.MoveComponent(
               MoveComponentRequest(
                   seat=seatLocation,
                   component=component,  # type: ignore
                   position=position,
               ),
               metadata=self.metadata,
           )
           return response

       async def CurrentPosition(self, row: int, index: int):
           response = await self._stub.CurrentPosition(
               CurrentPositionRequest(row=row, index=index),
               metadata=self.metadata,
           )
           return response
   ```

   Some important remarks about the wrapping `SeatService` class
   shown above:

   - The `SeatService` class must derive from the `Service` class provided by the Python SDK.
   - The `SeatService` class must use the grpc channel from the `Service` base class and provide it to the `_stub` in the `__init__` method. This allows the SDK to manage the physical connection to the grpc service and use service discovery of the middleware.
   - Every method needs to pass the metadata from the `Service` base class to the gRPC call. This is done by passing the `self.metadata` argument to the metadata of the gRPC call.

## Distributing your Python Vehicle Model

Now you a have a Python package containing your first Python _Vehicle Model_ and it is time to distribute it. There is nothing special about the distribution of this package, since it is just an ordinary Python package. Check out the [Python Packaging User Guide](https://packaging.python.org/en/latest/) to learn more about packaging and package distribution in Python.

### Distribute to single Vehicle App

If you want to distribute your Python _Vehicle Model_ to a single _Vehicle App_, you can do so by copying the entire folder `my_vehicle_model` under the `src` folder of your _Vehicle App_ repository and treat is a sub-package of the _Vehicle App_.

1. Create a new folder `my_vehicle_model` under `src` in your _Vehicle App_ repository.
2. Copy the `my_vehicle_model` folder to the `src` folder of your _Vehicle App_ repository.
3. Import the package `my_vehicle_model` in your _Vehicle App_:

```python
from <my_app>.my_vehicle_model import vehicle

...

my_app = MyVehicleApp(vehicle)
```

### Distribute inside an organization

If you want to distribute your Python _Vehicle Model_ inside an organization and use it to develop multiple _Vehicle Apps_, you can do so by creating a dedicated Git repository and copying the files there.

1. Create new Git repository called `my_vehicle_model`
2. Copy the content under `my_vehicle_model` to the repository.
3. Release the _Vehicle Model_ by creating a version tag (e.g., `v1.0.0`).
4. Install the _Vehicle Model_ package to your _Vehicle App_:

   ```python
   pip3 install git+https://github.com/<yourorg>/my_vehicle_model.git@v1.0.0
   ```

5. Import the package `my_vehicle_model` in your _Vehicle App_ and use it as shown in the previous section.

### Distribute publicly as open source

If you want to distribute your Python _Vehicle Model_ publicly, you can do so by creating a Python package and distributing it on the [Python Package Index (PyPI)](https://pypi.org/). PyPi is a repository of software for the Python programming language and helps you find and install software developed and shared by the Python community. If you use the `pip` command, you are already using PyPI.

Detailed instructions on how to make a Python package available on PyPI can be found [here](https://packaging.python.org/tutorials/packaging-projects/).

## Further information

- Concept: [Python SDK Overview](/docs/concepts/python_vehicle_app_sdk_overview.md)
- Tutorial: [Setup and Explore Development Enviroment](/docs/tutorials/setup_and_explore_development_environment.md)
- Tutorial: [Create a Python Vehicle App](/docs/tutorials/tutorial_how_to_create_a_vehicle_app.md)