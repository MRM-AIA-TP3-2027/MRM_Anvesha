CASHIER SYSTEM USING ROS2 (C++)

Overview:
This task implements a distributed cashier and inventory management system using ROS2. The system simulates a billing workflow where transactions are generated, processed, and reflected across multiple nodes.

The architecture consists of three primary nodes:
    • A bill generator node that produces transaction data
    • An inventory node that processes transactions and maintains system state
    • A status node that retrieves and displays the current system state

The system demonstrates inter-node communication using ROS2 topics and services, along with custom message and service definitions.

CONCEPTS IMPLEMENTED
    • ROS2 Nodes
    • Publish/Subscribe communication
    • Services (request-response model)
    • Custom message definitions(.msg)
    • Custom service definitions(.srv)
    • Cmake- based build system (ament)
    • Asynchronous communication patterns
    • Debugging and error resolution in ROS2


WORKSPACE SETUP:

1. Sourcing ROS2 Environment: source opt/ros/humble/setup.bash
2. Create Workspace:  mkdir -p ~/cashier_ws/src
			cd ~/cashier_ws
			colcon build
3. Source workspace: source install/setup.bash

Workspace structure:   cashier_ws/
			 ├── src/
			 ├── build/
 			 ├── install/
 			 ├── log/

4. Package creation: ros2 pkg create cashier_system –build-type ament_cmake –dependencies rclcpp std_msgs


CUSTOM INTERFACES:
1. Message Definition: msg/Bill.msg
			string item_name
			int32 quantity
			float32 price
2. Service Definition: srv/GetStatus.srv
		        ---
		         string status
BUILD CONFIGURATION

1. CmakeLists.txt

a) Add required dependencies:

find_package(rosidl_default_generators REQUIRED)

b) generate interfaces:

rosidl_generate_interfaces(${PROJECT_NAME}
“msg/Bill.msg”
“srv/GetStatus.srv”
)

c) export runtime dependencies:

ament_export_dependencies(rosidl_default_runtime)


2. package.xml:

<build_depend>rosidl_default_generators</build_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>

Build and verification
colcon build
source install/setup.bash

verify interfaces:
ros2 interface list | grep cashier_system

expected output:
cashier_system/msg/Bill
cashier_system/srv/GetStatus

SYSTEM IMPLEMENTATION:

1. Bill generator Node:
    • Accepts user input (item name, quantity, price)
    • Constructs a Bill message
    • Publishes messages to the /bill topic at regular intervals
it is the publisher with time based execution and handles the standard input.

2. Inventory Node:
    • Subscribes to all the /bill topics
    • Updates inventory and total income
    • Provides a service /get_status to return system state
    • std::map<std::string, int> for inventory tracking
    • floating point variable for total income
    • decrements inventory and updates the total income as quantity*price
3. Status Node
    • Acts as a service client
    • Periodically requests system status from the inventory node
    • Displays the current inventory and total income
    • Implementation uses asynchronous service calls to avoid deadlock
    • bill_generator → /bill → inventory _node
    • status_node → /get_status → inventory_node

RUNNING THE SYSTEM:
Open 3 terminals and source the environment in each:
source /opt/ros/humble/setup.bash
source ~/cashier_ws/install/setup.bash

Terminal 1
ros2 run cashier_system inventory_node
Terminal 2
ros2 run cashier_system bill_generator
Terminal 3
ros2 run cashier_system status_node

<img width="731" height="270" alt="image" src="https://github.com/user-attachments/assets/a26986f0-8737-470e-868e-94ae034d3ee2" />
<img width="731" height="270" alt="image" src="https://github.com/user-attachments/assets/2ead5cef-70f7-428e-951b-1c2e7a212e1e" />
<img width="736" height="317" alt="image" src="https://github.com/user-attachments/assets/3390ef99-edc8-4d76-90c2-8c2e19c673e3" />



RQT_GRAPH

<img width="1832" height="516" alt="image" src="https://github.com/user-attachments/assets/78b86909-7a08-4a34-a8bf-2b0fb32ee8bb" />


BUILD ERRORS:

Cause: Incorrect ordering of CMake commands
Resolution: Ensured add_executable appears before install


SERVICE CALL TIMEOUT
Cause: Blocking call using future.get() inside a timer callback
Resolution: Replaced with asynchronous callback-based request handling



