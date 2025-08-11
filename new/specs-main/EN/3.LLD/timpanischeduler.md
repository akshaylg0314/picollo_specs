# TIMPANI Scheduler (SW Center is in charge of)

## 1. Introduction

### Major features

TIMPANI Scheduler is a component that manages time-deterministic task execution in the PICCOLO framework.

1. Provides deterministic scheduling including periodic task scheduling, deadline-based execution, and jitter minimization.
2. Manages CPU resources through CPU core allocation and isolation, interrupt routing optimization, and real-time priority management.
3. Provides timing analysis capabilities for execution time monitoring, timing violation detection, and performance analysis and optimization.
4. Isolates safety-critical tasks, applies priority inheritance and ceiling mechanisms, and ensures deterministic behavior.
5. Supports various real-time scheduling algorithms to meet different workload requirements.
6. Utilizes and extends the real-time scheduling capabilities of the Linux kernel when necessary.

### Main Dataflow

1. Receives container scheduling parameters from ActionController.
2. Sets up CPU isolation and interrupt routing.
3. Plans and manages execution schedules for periodic tasks.
4. Monitors task execution times and deadline compliance.
5. Collects performance data and provides it to MonitoringServer.
6. Sends notifications to ApiServer in case of timing violations.

## 2. Reference information

### 2.1 Related components
- **ActionController**: Forwards task execution requests to TIMPANI scheduler
- **APIServer**: Receives timing violation and scheduler status notifications
- **MonitoringServer**: Collects scheduler performance metrics

### 2.2 Protocols and APIs
- **gRPC API**: gRPC services for communication with ActionController and APIServer
- **Kernel Scheduling API**: Linux real-time scheduling interfaces
- **cgroup API**: cgroups interfaces for CPU resource control
- **Monitoring API**: Interfaces for performance metrics collection

### 4.4 References
- [Linux Real-time Scheduling Documentation](https://man7.org/linux/man-pages/man7/sched.7.html)
- [SCHED_DEADLINE Documentation](https://www.kernel.org/doc/html/latest/scheduler/sched-deadline.html)
- [cgroups v2 Documentation](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)
- [CPU Isolation Techniques](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/performance_tuning_guide/sect-red_hat_enterprise_linux-performance_tuning_guide-cpu-isolating_cpus)
- [Task Scheduling Theory](https://en.wikipedia.org/wiki/Scheduling_(computing))
