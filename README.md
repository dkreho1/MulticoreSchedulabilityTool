# Multi-core Schedulability and Allocation Tool
This multi-core schedulability analysis and allocation tool has been developed as part of the authors' Master's Thesis in Intelligent Embedded Systems at Mälardalen University (Sweden) titled: "Distributed functionality allocation to multi-core computational nodes"
Authors: Enesa Hrustić & Dženan Kreho  
Contact: {ehc23002 | dko23001}@student.mdu.se

## User guide
The purpose of the tool is to evaluate the feasibility of a task set on a specified multi-core system, using the schedulability types, interconnect variants, and arbitration policies discussed in the thesis. The following text provides a guide on how to use the tool and specify the input.

The input is specified through a .txt file. The file is organized into three sections: configuration parameters, output options, and task set specification. Configuration includes the following parameters:
 - Number of cores: This parameter specifies the number of cores in the multi-core system
 - Interconnect: Parameters in this subset specify the interconnect type and arbitration policy:
	 - Type: Bus or Crossbar
	 - Arbitration Policy: FCFS or RR
	 - Slot size: If RR arbitration is selected, the slot size (time quantum) must also be specified.
	 - Number of banks: If the crossbar is used as the interconnect type, the number of banks into which the main memory is divided must also be specified.
	 - Task-to-bank mapping: True or False
		 - True specifies that each task will have its bank assigned within the input file
		 - False specifies that the tool should randomly map tasks to banks
 - Scheduling:
	 - Policy: Preemptive or Non-preemptive
	 - Interrupt priority: Specifies the priority of interrupts in the system. This parameter should indicate the highest priority in the task set.
	 - Hyperperiod: Specifies the hyperperiod of the task set
	 - Inverse Priority Logic: True or False
		 - True specifies that lower numbers indicate higher priorities
		 - False specifies that higher numbers indicate higher priorities
 - Task phases: WCET or MD value or MD range
	 - WCET indicates that the WCET of all three task phases will be provided together with the task set
	 - MD value specifies the value of memory demand parameter for all tasks
	 - MD range specified as [min\_value, max\_value], the tasks will have randomly assinged MD value from the specified range
 - Allocation:
	 - Heuristics: Specifies which heuristic(s) are used for task-to-core mapping. Can take values: First fit or Best fit or Worst fit. In case when two or three heuristics will be used, they should be separated with comma and space, e.g.: First fit, Best fit, Worst fit
	 - Number of reallocations per heuristic: Specifies the number of trials that the tool will perform in order to find feasible mapping of tasks to cores. Each specified heuristic will be performed this number of times, until the tool finds a schedulable solution
	 - Core utilization limit: Specifies the utilization limit for each core in the system. The maximum value this parameter can take is 1.

For the output options only parameter needs to be specified:
 - List unschedulable tasks: True or False
	 - True indicates that if the task set is not schedulable using the given configuration, the output file will include the names of the unschedulable tasks along with their WCRT.
	 - False specifies that the output file will only say that the task set is not schedulable. Specifying this option allows the tool to run faster.

Once all of the necessary configurations and options are provided, the parameters regarding tasks and applications need to be specified:
 - Number of applications: Specifies the number of applications that tasks are assigned to. In case tasks are not mapped to applications, this parameter should be set to 1.
 - Applications: In this section of the tool input, the task sets per application should be provided. Each task set should include:
	 - Application name
	 - Application criticality: If the number of applications is set to 1, this parameter will not be significant.
	 - Standalone: True or False. This parameter specifies whether the current application should completely be allocated to one core (True) or not (False)
	 - Number of tasks: This parameter specifies the number of tasks in the application
 - Tasks: The parameters that should be provided along with the tasks depend on the chosen configuration settings. A few cases can be derived:
	 - Case 1: If the memory demand values (MD) are specified as a single value or range of values, the task set should include (in the following format): Task Name: Priority WCET Period Deadline
	 - Case 2: If WCET option is chosen in the Task phases configuration parameter then each task in the task set should include: Task Name: Priority WCET\_Read WCET\_Execute WCET\_Write Period Deadline
	 - Case 3: When the crossbar is selected as the interconnect type and the task-to-bank mapping is known, the bank number parameter should be specified at the end of the previous two task set formats. For example: Priority WCET Period Deadline Bank
	 - Case 4: If the task set consists of interrupts as well, beside all parameters that were described in the 3 cases above, interrupts should be specified with (Interrupt) before their name, e.g., (Interrupt) Task Name: Priority WCET Period Deadline

It worth noting that the configurations and specifications presented earlier should be provided in the same order. Additionally, comments are permitted at the top of the .txt file, indicated by "//". Multiple comment lines are allowed, but they must all be at the beginning of the file.

To enable better management of errors due to improper file formatting, a set of error codes was used for error indication. The values along with code names (meanings), are specified in the following table:
| Name | Value |
| :---: | :---: |
|UNABLE TO OPEN FILE|2|
|INVALID FILE FORMAT|3|
|INVALID INTERCONNECT OPTION|4|
|INVALID ARBITRATION POLICY|5|
|INVALID SCHEDULING POLICY|6|
|INVALID TASKS TO CORE OPTION|7|
|INVALID TASK PHASES OPTION|8|
|INVALID HEURISTIC|9|
|INVALID OUTPUT OPTION|10|
|INVALID APP STANDALONE OPTION|11|
|INVALID BANK NUMBER|12|
|INVALID PRIORITY LOGIC OPTION|13|

The tool itself can be run through the terminal. The program accepts two optional command line arguments. The first is the path to the input file, while the second is the path to the output file. If no arguments are provided, then the input will be loaded from "input.txt" and the output will be written to "output.txt".

The remainder of the user guide provides a few examples of input .txt files.
Example 1:

    // Example: Crossbar, FCFS, Random bank assignment, Preemptive scheduling,
    //          Task phases defined using one MD, All three heuristics
    Configuration:
    	Number of cores: 2
    	Interconnect:
    		Type: crossbar
    		Arbitration policy: FCFS
    		Number of banks: 4
    		Task-to-bank mapping: false
    	Scheduling:
    		Policy: Preemptive
    		Hyperperiod: 1000000
    		Inverse Priority Logic: true
    	Task phases: MD = 0.1
    	Allocation:
    		Heuristics: First fit, Worst fit, Best fit
    		Number of reallocations per heuristic: 10
    		Core utilization limit: 0.875
    Output:
    	List unschedulable tasks: false
    Applications:
    	Number of applications: 2
    	Applications:
                Application1:
                    Criticality: 0
                    Standalone: false
                    Number of tasks: 2
                    Tasks:
                        Task0: 0 1 10 10
                        Task1: 1 3 20 20
                Application2:
                    Criticality: 1
                    Standalone: true
                    Number of tasks: 3
                    Tasks:
                        Task2: 0 3 10 10
                        (Interrupt) Task3: 1 5 20 20
                        Task4: 2 4 15 15

Example 2:

    // Example: Crossbar, FCFS, Banks manually assigned, Non-preemptive scheduling,
    //          Task phases defined using MD range, All three heuristics
    Configuration:
    	Number of cores: 2
    	Interconnect:
    		Type: crossbar
    		Arbitration policy: FCFS
    		Number of banks: 4
    		Task-to-bank mapping: true
    	Scheduling:
    		Policy: Non-preemptive
    		Hyperperiod: 1000000
    		Inverse Priority Logic: true
    	Task phases: MD = [0.05, 0.1]
    	Allocation:
    		Heuristics: First fit, Worst fit, Best fit
    		Number of reallocations per heuristic: 10
    		Core utilization limit: 0.875
    Output:
    	List unschedulable tasks: false
    Applications:
    	Number of applications: 2
    	Applications:
                Application1:
                    Criticality: 0
                    Standalone: false
                    Number of tasks: 2
                    Tasks:
                        Task0: 0 1 10 10 0
                        Task1: 1 3 20 20 1
                Application2:
                    Criticality: 1
                    Standalone: true
                    Number of tasks: 3
                    Tasks:
                        Task2: 0 3 10 10 0
                        (Interrupt) Task3: 1 5 20 20 3
                        Task4: 2 4 15 15 2
Example 3:

    // Example: Bus, RR, Preemptive scheduling,
    //          Task phases defined using WCETs, One heuristic
    Configuration:
    	Number of cores: 2
    	Interconnect:
    		Type: bus
    		Arbitration policy: RR
            Slot size: 1
    	Scheduling:
    		Policy: Preemptive
    		Hyperperiod: 1000000
    		Inverse Priority Logic: true
    	Task phases: WCET
    	Allocation:
    		Heuristics: Worst fit 
    		Number of reallocations per heuristic: 10
    		Core utilization limit: 0.875
    Output:
    	List unschedulable tasks: true
    Applications:
    	Number of applications: 1
    	Applications:
                Application1:
                    Criticality: 0
                    Standalone: false
                    Number of tasks: 2
                    Tasks:
                        Task0: 0 1 1 1 10 10
                        Task1: 1 3 2 1 20 20