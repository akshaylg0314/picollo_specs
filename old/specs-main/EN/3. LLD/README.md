# PICCOLO (PULLPIRI) Software Design Documents

This directory contains detailed design documents for each component of the PICCOLO(PULLPIRI) framework.

## Project Overview

PICCOLO is the internal code name for the Eclipse PULLPIRI project, an OS-independent container framework optimized for vehicle environments. PULLPIRI is a vehicle system orchestrator framework that aims to apply the benefits of cloud-native technology to in-vehicle services and applications, providing integrated orchestration of both resources and services in vehicle environments.

## Component Documents

### System Management Components
- [`statemanager.md`](statemanager.md) - StateManager that manages and coordinates the state of systems and services
- [`settingsservice.md`](settingsservice.md) - SettingsService that provides UI and CLI for system settings management
- [`policymanager.md`](policymanager.md) - PolicyManager that determines service priorities and decides allow/wait/deny

### Execution Management Components
- [`actioncontroller.md`](actioncontroller.md) - ActionController that manages container lifecycle
- [`apiserver.md`](apiserver.md) - APIServer responsible for providing APIs and registering/preparing Artifacts
- [`filtergateway.md`](filtergateway.md) - FilterGateway that filters and routes service requests

### Monitoring and Node Management Components
- [`monitoringserver.md`](monitoringserver.md) - MonitoringServer responsible for metrics collection and analysis
- [`nodeagent.md`](nodeagent.md) - NodeAgent that monitors and manages the state of each node

### Networking and Scheduling Components
- [`timpanischeduler.md`](timpanischeduler.md) - TIMPANI Scheduler that provides scheduling for real-time applications
- [`pharosnetworking.md`](pharosnetworking.md) - Component providing eBPF-based network orchestration functionality

## Recent Architecture Changes

### Common ETCD Library Integration (2025-07-25)
We've integrated individual ETCD client implementations used in all components into a common library in the `common/etcd` package.
This reduces code duplication, provides a consistent ETCD integration method, and makes future ETCD-related feature expansion and maintenance easier.

**Major Changes:**
- Standardization of ETCD client connection, read/write, and watch functions
- Providing integrated interfaces for JSON/YAML serialization/deserialization
- Enhanced transaction and batch processing capabilities
- Standardized connection error and retry handling logic
- Performance optimization and memory usage improvement

**Changed Files:**
- `statemanager` - `etcd/client.rs` → Using `common/etcd` library
- `settingsservice` - `etcd/client.rs` → Using `common/etcd` library
- `policymanager` - `etcd/client.rs` → Using `common/etcd` library
- `actioncontroller` - `storage/etcd.rs` → Using `common/etcd` library
- `apiserver` - `artifact/data.rs` ETCD-related functions → Using `common/etcd` library
- `monitoringserver` - `metrics/store.rs` ETCD-related functions → Using `common/etcd` library

### Cloud Communication Architecture Improvement (2025-07-25)
Removed CloudManager references from `monitoringserver` and separated cloud communication functionality into an independent service.

**Major Changes:**
- Created new CloudConnector service for cloud communication
- Implemented event-based communication between MonitoringServer and CloudConnector
- Added configurable message queuing with reliability guarantees
- Enhanced offline handling and synchronization upon reconnection
- Improved error handling and retry logic

**Implementation Impact:**
- Reduced coupling between monitoring and cloud communication
- Improved system stability during network outages
- Added capability to prioritize different types of cloud messages
- Enhanced logging and diagnostics for cloud communication issues
- Simplified future cloud integration expansion

## Development Guidelines

### Coding Standards
All components should follow the project's coding standards:
- Error handling through Result type with descriptive error messages
- Comprehensive logging with appropriate log levels
- Unit tests for all public functions
- Integration tests for component interfaces
- Documentation for all public APIs and configurations

### Dependency Management
- Use versioned dependencies with explicit versions in Cargo.toml
- Minimize external dependencies when possible
- Prefer crates from the Rust official ecosystem
- Review security implications of all dependencies

### Performance Considerations
- Consider resource constraints of embedded environments
- Optimize for startup time and runtime performance
- Implement appropriate caching strategies
- Use async/await for I/O-bound operations
- Profile and benchmark performance-critical paths
