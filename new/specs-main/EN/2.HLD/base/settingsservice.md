# Settings Service Detailed Design Document

**Document Number**: PICCOLO-SETTINGS-2025-001  
**Version**: 1.0  
**Date**: 2025-08-05  
**Author**: PICCOLO Team  
**Category**: HLD (High-Level Design)

## 1. Overview

The Settings Service is a core component for system configuration management in the PICCOLO framework. It provides various interfaces for administrators and developers to easily manage system configuration, including web-based UI and CLI for YAML-based configuration file creation, editing, verification, and deployment. It also supports configuration change history management and rollback features.

### 1.1 Purpose

The main purposes of the Settings Service are as follows:

1. Provide centralized configuration management for various components in the PICCOLO framework
2. Enable easy and safe application of configuration changes through multiple interfaces
3. Ensure system stability through change history management and rollback
4. Restrict configuration access based on user permissions
5. Support unified configuration creation using templates

### 1.2 Main Features

The Settings Service provides the following main features:

1. **Configuration Management**
   - Create, edit, and delete YAML-based configuration files
   - Validate configuration and analyze impact
   - Apply changes and monitor results

2. **Multiple Interfaces**
   - Web-based UI
   - Powerful CLI interface
   - REST API and gRPC interface

3. **Change Management**
   - Save and view change history
   - Support rollback to previous versions
   - Change verification and collision prevention

4. **User Management**
   - Permission-based access control
   - User authentication and authorization
   - Change audit logging

5. **Monitoring Management**
   - Resource-specific monitoring item management
   - Add/delete monitoring items
   - Dashboard configuration

## 2. Architecture

The Settings Service is designed with a modular structure that clearly separates each functional area. This improves maintainability and scalability, allowing for independent development and testing of individual components.

### 2.1 Main Components

The main components of the Settings Service are as follows:

1. **Core Manager**
   - Service initialization and component coordination
   - Process configuration change requests and control flow
   - Manage interactions between other modules

2. **Authentication & Authorization**
   - User authentication and session management
   - Permission-based access control
   - User and role management

3. **Configuration Manager**
   - Load and save configuration files
   - Configuration validation
   - Change impact analysis

4. **History Manager**
   - Configuration change history management
   - Version comparison
   - Rollback functionality

5. **Monitoring Manager**
   - Monitoring item management
   - Resource-specific monitoring configuration
   - Monitoring data collection

6. **Web Interface**
   - Web server and API
   - User interface components
   - Real-time configuration editor

7. **CLI Interface**
   - Command-line command processing
   - Interactive shell
   - Batch processing

8. **External Integration**
   - API Server integration
   - ETCD storage integration
   - External system integration

### 2.2 Data Flow

The main data flows in the Settings Service are as follows:

1. **Configuration Change Flow**
   - User requests configuration change via Web UI or CLI
   - Request authentication and permission verification
   - Configuration validation and impact analysis
   - Save changes to ETCD and record history
   - Forward changes to API Server
   - Monitor change application results and provide feedback

2. **Rollback Flow**
   - User requests rollback to previous version
   - Request authentication and permission verification
   - Load previous version configuration
   - Create and validate rollback plan
   - Execute rollback and verify results
   - Record rollback history

3. **Monitoring Flow**
   - User requests adding/removing monitoring items
   - Request authentication and permission verification
   - Validate monitoring configuration
   - Save and apply monitoring settings
   - Start/stop monitoring data collection
   - Visualize monitoring results

### 2.3 System Integration

The Settings Service integrates with other PICCOLO framework components as follows:

1. **API Server**
   - Forward changed configuration
   - Check configuration application status
   - Synchronize system-wide configuration

2. **State Manager**
   - Check system state after configuration changes
   - Monitor component status
   - Make rollback decisions based on state

3. **ETCD**
   - Store configuration data
   - Manage change history
   - Configuration version management

4. **Monitoring Server**
   - Forward monitoring configuration
   - Collect monitoring data
   - Create and manage alerts

## 3. Interfaces

The Settings Service provides various user and system interfaces.

### 3.1 User Interfaces

#### 3.1.1 Web UI

The web-based user interface provides the following features:

1. **Dashboard**
   - System configuration overview
   - Recent changes
   - Status monitoring
   - Monitoring metrics visualization

2. **Configuration Editor**
   - Hierarchical resource browsing
   - YAML/JSON editor
   - Schema-based validation
   - Property-based form editing

3. **History Browser**
   - View configuration change history
   - Compare versions
   - Rollback functionality

4. **Monitoring Management**
   - PICCOLO resource monitoring item list
   - Add/remove monitoring items
   - Configure monitoring dashboard

5. **User Management**
   - User and role management
   - Permission settings
   - Audit log viewing

#### 3.1.2 CLI

The command-line interface provides the following features:

1. **Basic Commands**
   - Get/set/delete configuration
   - Apply configuration
   - Validate configuration

2. **History Management**
   - View change history
   - Compare versions
   - Rollback to previous versions

3. **Interactive Shell**
   - Auto-completion
   - Command history
   - Inline help

4. **Batch Processing**
   - Script execution
   - Pipeline integration
   - Automation support

5. **Monitoring Management**
   - View monitoring items
   - Add/remove monitoring items
   - Check monitoring status

### 3.2 System Interfaces

#### 3.2.1 REST API

The REST API provides the following endpoints:

1. **Configuration Management**
   - `GET /api/config/{key}` - Get configuration
   - `PUT /api/config/{key}` - Change configuration
   - `DELETE /api/config/{key}` - Delete configuration
   - `POST /api/config/apply` - Apply configuration
   - `POST /api/config/validate` - Validate configuration

2. **History Management**
   - `GET /api/history` - View history
   - `GET /api/history/{id}` - View specific history entry
   - `POST /api/rollback` - Execute rollback

3. **Monitoring Management**
   - `GET /api/monitoring/resources` - List of monitorable resources
   - `GET /api/monitoring/items/{resource}` - View resource-specific monitoring items
   - `POST /api/monitoring/items` - Add monitoring item
   - `DELETE /api/monitoring/items/{id}` - Delete monitoring item

4. **User Management**
   - `POST /api/auth/login` - Login
   - `POST /api/auth/logout` - Logout
   - `GET /api/users` - User list
   - `POST /api/users` - Create user

#### 3.2.2 gRPC API

The gRPC API provides the following services:

1. **ConfigService**
   - `GetConfig` - Get configuration
   - `SetConfig` - Change configuration
   - `ApplyConfig` - Apply configuration
   - `ValidateConfig` - Validate configuration

2. **HistoryService**
   - `ListHistory` - View history
   - `GetHistoryEntry` - View specific history entry
   - `RollbackConfig` - Execute rollback

3. **MonitoringService**
   - `ListMonitoringResources` - List of monitorable resources
   - `GetMonitoringItems` - View resource-specific monitoring items
   - `AddMonitoringItem` - Add monitoring item
   - `DeleteMonitoringItem` - Delete monitoring item

## 4. Data Models

The main data models used in the Settings Service are as follows.

### 4.1 Configuration Data

```

### 4.3 Monitoring Item

```yaml
# Monitoring item example
id: monitoring-123456
name: "CPU Usage Monitoring"
resourceType: "node"
resourceId: "node-01"
metricType: "cpu_usage"
thresholds:
  warning: 70
  critical: 90
interval: 60
unit: "percentage"
enabled: true
notificationChannels:
  - email
  - slack
createdBy: "admin"
createdAt: "2025-08-05T13:45:00Z"
```

### 4.4 User Information

```yaml
# User information example
id: user-123456
username: admin
email: admin@example.com
roles:
  - admin
  - configurator
permissions:
  - config.read
  - config.write
  - config.apply
  - history.read
  - history.rollback
lastLogin: "2025-08-05T10:15:00Z"
status: ACTIVE
```

## 5. Detailed Functionality

### 5.1 Configuration Management

#### 5.1.1 Configuration Validation

Configuration validation is performed in the following steps:

1. **Syntax Validation**: Check for YAML/JSON syntax errors
2. **Schema Validation**: Validate against defined schema
3. **Semantic Validation**: Validate semantic validity of configuration values
4. **Dependency Validation**: Check dependencies with other configurations
5. **Impact Analysis**: Analyze impact of changes on the system

#### 5.1.2 Configuration Application

Configuration application is performed in the following steps:

1. **Pre-validation**: Validate before application
2. **Change Planning**: Plan changes to be applied
3. **Backup**: Backup current configuration
4. **Application**: Apply changes through API Server
5. **Monitoring**: Monitor application results
6. **Confirm/Rollback**: Confirm success or rollback if issues occur

### 5.2 History Management

#### 5.2.1 Change History Tracking

All configuration changes are recorded with the following information:

1. **Time Information**: Change date and time
2. **User Information**: User who performed the change
3. **Change Content**: Specific changes made
4. **Version Information**: Before and after versions
5. **Change Reason**: User-provided explanation

#### 5.2.2 Rollback Functionality

The rollback functionality is performed in the following process:

1. **Target Version Selection**: Select previous version to roll back to
2. **Rollback Plan Creation**: Create change plan from current state to target state
3. **Impact Analysis**: Analyze impact of rollback on the system
4. **Rollback Execution**: Perform rollback according to plan
5. **Result Verification**: Verify rollback success
6. **History Recording**: Record rollback operation in history

### 5.3 Monitoring Management

#### 5.3.1 Monitoring Item Management

Monitoring item management provides the following functions:

1. **View Resource-specific Monitoring Items**: Provide list of monitoring items by PICCOLO resource type
2. **Add Monitoring Items**: Create and configure new monitoring items
3. **Delete Monitoring Items**: Remove unnecessary monitoring items
4. **Enable/Disable Monitoring Items**: Manage monitoring item status

#### 5.3.2 Resource Monitoring Configuration

Resource monitoring configuration is performed in the following steps:

1. **Resource Selection**: Select PICCOLO resource to monitor
2. **Metric Selection**: Select metrics supported by the resource
3. **Threshold Setting**: Set warning and critical thresholds
4. **Collection Interval Setting**: Set data collection frequency
5. **Notification Channel Setting**: Set channels to receive alerts when thresholds are exceeded
6. **Save/Apply Settings**: Save and apply monitoring settings

### 5.4 User Management

#### 5.4.1 Authentication and Authorization

Authentication and authorization are managed as follows:

1. **User Authentication**: Username/password or token-based authentication
2. **Session Management**: Login session management
3. **Permission Model**: Role-based access control (RBAC)
4. **Permission Verification**: Verify permissions before performing actions
5. **Audit Logging**: Record all authentication and permission-related activities

#### 5.4.2 User Management

User management provides the following functions:

1. **User Registration**: Add new users
2. **User Information Management**: Edit user information
3. **Role Assignment**: Assign roles to users
4. **Permission Management**: Set permissions by role
5. **User Status Management**: Manage active/inactive status

## 6. Security Considerations

The Settings Service is designed with the following security aspects in mind:

1. **Authentication and Authorization**
   - Strong authentication system
   - Granular permission model
   - Principle of least privilege

2. **Data Protection**
   - Encryption of sensitive configuration information
   - Protection of data in transit (TLS)
   - Protection of stored data

3. **Audit and Tracking**
   - Record all changes
   - Log user activities
   - Protect audit logs

4. **Input Validation**
   - Validate all user inputs
   - Prevent malicious inputs
   - Safe parsing and processing

5. **API Security**
   - Restrict API access
   - Rate limiting
   - Token-based authentication

## 7. Performance Considerations

The Settings Service is designed with the following performance aspects in mind:

1. **Responsiveness**
   - Optimize user interface response time
   - Improve responsiveness through asynchronous processing
   - Provide user feedback

2. **Scalability**
   - Support distributed architecture
   - Handle large configuration datasets
   - Support simultaneous access by multiple users

3. **Resource Usage**
   - Efficient memory management
   - Optimize CPU usage
   - Storage space efficiency
   - Caching

## 8. Implementation Considerations

The following considerations should be taken into account when implementing the Settings Service:

1. **Technology Stack**
   - Rust language
   - Actix Web framework
   - gRPC communication
   - ETCD storage

2. **Modularity**
   - Separate modules by functionality
   - Clear interface definitions
   - Independent testing of modules

3. **Testability**
   - Structure conducive to unit testing
   - Support for integration testing
   - Use of mock objects

4. **Documentation**
   - In-code documentation
   - API documentation
   - User guide

5. **Internationalization**
   - Multi-language support
   - Localization settings
   - Cultural expression considerations

## 9. Deployment and Operations

Deployment and operational considerations for the Settings Service are as follows:

1. **Containerization**
   - Provide Docker image
   - Support Kubernetes deployment
   - Configurable environment variables

2. **Monitoring**
   - Status monitoring
   - Performance metrics collection
   - Log aggregation

3. **Backup and Recovery**
   - Regular configuration data backup
   - Failure recovery procedures
   - Data consistency maintenance

4. **Upgrades**
   - Support for zero-downtime upgrades
   - Compatibility with previous versions
   - Rollback plans

5. **Operational Documentation**
   - Installation guide
   - Operations manual
   - Troubleshooting guide

## 10. Conclusion

The Settings Service is a core component of the PICCOLO framework, providing various interfaces and functions for efficient and safe management of system configurations. It enhances system stability and user productivity by offering intuitive user experiences through Web UI and CLI, along with powerful validation, history management, and monitoring management capabilities.

This design document provides an overview of the basic architecture and main features of the Settings Service, with details that may be added or adjusted during the actual implementation process. Ultimately, the Settings Service supports PICCOLO framework components to be configured, managed, and monitored in a consistent and reliable manner.

### 4.2 History Entry

```yaml
# History entry example
id: history-123456
timestamp: "2025-08-05T14:30:00Z"
user: admin
action: UPDATE
configKey: example-config
version: "v1.2.3"
changes:
  - path: spec.parameter1
    oldValue: oldValue1
    newValue: value1
  - path: spec.nestedConfig.subParam2
    oldValue: oldSubValue2
    newValue: subValue2
comment: "Updated network parameters"
status: APPLIED
```
