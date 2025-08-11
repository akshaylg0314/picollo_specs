# PICCOLO (PULLPIRI) Development Roadmap

**Document Information:**
- Document ID: PICCOLO-DEV-ROADMAP-2025
- Version: 1.0
- Date: 2025-07-28
- Classification: Development Plan

## Overview

This document defines the detailed development schedule for the PICCOLO(PULLPIRI) project. It outlines future development phases based on the current initial implementation status in the GitHub repository ([eclipse-pullpiri/pullpiri](https://github.com/eclipse-pullpiri/pullpiri/tree/refactoring)).

## Current Development Status Analysis

### Completed Components and Features
- **Common Library Implementation**:
  - Basic ETCD client implementation and integration
  - YAML parsing and validation functionality
  - Basic gRPC protocol definitions

- **Partially Implemented Core Components**:
  - **APIServer**: Basic REST API structure and YAML parsing implementation
  - **StateManager**: Basic state management functionality
  - **ActionController**: Rudimentary container runtime adapters

### Unimplemented or Initial Stage Components
- **FilterGateway**: Scenario processing and policy application functionality
- **PolicyManager**: Policy definition and application mechanisms
- **MonitoringServer**: Metrics collection and analysis
- **NodeAgent**: Node status monitoring
- **TIMPANI Scheduler**: Real-time scheduling
- **PHAROS Networking**: Network orchestration
- **SettingsService**: Settings management interface

## Development Phase Plan

### Phase 1: Core Component Completion (2025-08 ~ 2025-09)

#### Goal
Complete the basic implementation of core components that provide fundamental orchestration capabilities to build a minimally viable system that manages both services and resources.

#### Key Tasks
1. **APIServer Enhancement** (2025-08-01 ~ 2025-08-15)
   - Complete REST API endpoints and strengthen security features
   - Enhance artifact validation and storage functionality
   - Implement notification system

2. **ActionController Expansion** (2025-08-01 ~ 2025-08-20)
   - Complete runtime adapter pattern (Podman, Docker, Containerd support)
   - Implement container lifecycle management functions
   - Implement resource limitation and allocation features

3. **StateManager Completion** (2025-08-10 ~ 2025-08-25)
   - Implement state transition and synchronization mechanisms
   - Enhance distributed state management functionality
   - Implement error recovery strategies

4. **FilterGateway Basic Implementation** (2025-08-15 ~ 2025-08-31)
   - Implement scenario processing logic
   - Implement basic filtering mechanisms
   - Integration with ActionController and PolicyManager

5. **Common Library Optimization** (2025-08-20 ~ 2025-09-05)
   - Extend ETCD client functionality and optimize performance
   - Improve serialization/deserialization framework
   - Expand logging and monitoring utilities

#### Deliverables
- System capable of basic container deployment and management with resource orchestration
- Minimal scenario processing capability
- Integrated test framework and basic test cases

### Phase 2: Extended Functionality Implementation (2025-09 ~ 2025-10)

#### Goal
Implement advanced features to support complex scenarios needed in actual vehicle environments and establish monitoring systems for both services and resources.

#### Key Tasks
1. **PolicyManager Implementation** (2025-09-01 ~ 2025-09-15)
   - Implement policy definition and management interface
   - Develop policy-based decision engine
   - Implement security and resource policies

2. **MonitoringServer Development** (2025-09-01 ~ 2025-09-20)
   - Implement metrics collection and processing system for both services and resources
   - Implement log aggregation and analysis functionality
   - Implement alerting and anomaly detection system
   - Integrate dashboards and visualization tools

3. **NodeAgent Development** (2025-09-10 ~ 2025-09-30)
   - Implement node status monitoring functionality
   - Implement local resource management and control features
   - Implement secure deployment mechanisms

4. **SettingsService Development** (2025-09-15 ~ 2025-10-05)
   - Implement UI and CLI interfaces
   - Implement settings management and change validation system
   - Implement rollback and version management features

5. **Integration and Stabilization** (2025-09-20 ~ 2025-10-10)
   - Strengthen integration testing between components
   - Improve error handling and recovery mechanisms
   - Performance optimization and scalability testing

#### Deliverables
- Policy-based orchestration system for both services and resources
- Comprehensive monitoring and alerting system
- User-friendly settings management interface
- Multi-node support and distributed system functionality

### Phase 3: Specialized Features and Performance Optimization (2025-10 ~ 2025-11)

#### Goal
Implement vehicle-specific features and optimize real-time performance to evolve the system for production environment use with advanced resource and service orchestration.

#### Key Tasks
1. **TIMPANI Scheduler Development** (2025-10-01 ~ 2025-10-20)
   - Implement deterministic real-time scheduling
   - Implement CPU core isolation and allocation mechanisms
   - Implement time constraint analysis and application functionality

2. **PHAROS Networking Development** (2025-10-01 ~ 2025-10-25)
   - Implement eBPF-based network orchestration
   - Implement packet filtering and routing functionality
   - Implement Quality of Service (QoS) guarantee mechanisms
   - Implement network isolation and security features

3. **Specialized Manager Implementation** (2025-10-15 ~ 2025-11-05)
   - Implement DeviceManager
   - Implement AudioManager
   - Implement WindowManager
   - Implement StorageManager
   - Implement NodeManager

4. **Cloud Integration Enhancement** (2025-10-20 ~ 2025-11-10)
   - Implement and integrate CloudManager
   - Implement OTA update mechanism
   - Implement remote monitoring and management functionality

5. **Performance Optimization and Scalability Improvement** (2025-10-25 ~ 2025-11-15)
   - Optimize resource usage
   - Improve startup and response times
   - Support large-scale deployment scenarios

#### Deliverables
- System optimized for vehicle environments with guaranteed real-time performance
- Integrated management functionality for specialized resources
- Cloud integration and OTA capabilities
- Performance-optimized and scalable system

### Phase 4: System Validation and Stabilization (2025-11 ~ 2025-12)

#### Goal
Ensure reliability in production environments through comprehensive testing and system stabilization work

#### Key Tasks
1. **Expand Integration Testing** (2025-11-01 ~ 2025-11-20)
   - Implement scenario-based E2E testing
   - Conduct load testing and stress testing
   - Perform testing in actual vehicle environments

2. **Security Enhancement** (2025-11-10 ~ 2025-11-25)
   - Analyze and address security vulnerabilities
   - Improve permission management system
   - Implement additional security features for safety-critical systems

3. **Complete Documentation** (2025-11-15 ~ 2025-12-05)
   - Complete API documentation
   - Create user manuals
   - Write developer guides
   - Create deployment and operations guides

4. **System Stabilization** (2025-11-20 ~ 2025-12-10)
   - Fix bugs and improve error handling
   - Perform performance tuning and optimization
   - Validate error recovery mechanisms

5. **Final Release Preparation** (2025-12-01 ~ 2025-12-15)
   - Deploy release candidate version
   - Conduct final testing and validation
   - Prepare packaging and deployment systems

#### Deliverables
- Fully tested and validated system
- Comprehensive documentation
- Production-ready release version
- Maintenance and support framework

## Key Milestones and Schedule

| Milestone | Expected Completion Date | Key Deliverables |
|-----------|--------------------------|------------------|
| **Phase 1 Completion** | 2025-09-10 | Core components with basic functionality implemented |
| **Phase 2 Completion** | 2025-10-15 | Extended functionality and monitoring system |
| **Phase 3 Completion** | 2025-11-20 | Specialized features and performance optimization completed |
| **Phase 4 Completion** | 2025-12-15 | Final release version |

## Resource Planning

### Required Personnel
- Core Developers: 5 (with Rust experience)
- UI/UX Developer: 1 (web and CLI interfaces)
- QA Engineers: 2
- System Engineer: 1 (deployment and infrastructure)
- Technical Writer: 1 (documentation)

### Hardware Requirements
- Development Servers: 2 high-performance development servers
- Test Environment: Vehicle emulation environment setup
- CI/CD Infrastructure: Continuous integration/deployment pipeline

## Risk Factors and Mitigation Strategies

### Technical Risks
- **Risk**: Difficulty ensuring real-time performance in vehicle environments
  - **Mitigation**: Monitor real-time performance metrics and automate testing from early stages

- **Risk**: Complexity in supporting various container runtimes
  - **Mitigation**: Design robust abstraction layers and develop comprehensive test cases

### Schedule-Related Risks
- **Risk**: Potential delays in specialized component development
  - **Mitigation**: Implement core functionality first through priority setting

- **Risk**: Unexpected issues during integration
  - **Mitigation**: Apply gradual integration and continuous testing

## Conclusion

The PICCOLO(PULLPIRI) project will implement functionality in four development phases, from basic core features to vehicle environment-specialized advanced features. This roadmap was created based on the initial implementation status of the current GitHub repository and will be continuously updated according to project progress.
