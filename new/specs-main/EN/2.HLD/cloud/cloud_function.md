# PICCOLO Cloud CI/CD Platform Specification

**Document Number**: PICCOLO-CLOUD-CICD-2025-002  
**Version**: 1.1  
**Date**: 2025-07-17  
**Author**: joshua-jung_LGESDV  
**Classification**: System Specification  
**Last Modified**: 2025-07-17 10:05:51 UTC

## 1. Overview

This document describes the detailed specification of the cloud CI/CD platform for developing, building, and deploying YAML-based applications in the PICCOLO system. It defines the required information for each stage, the deliverables, and the data passed to the next stage.

### 1.1 Purpose

- Provide an integrated platform for development, testing, building, and deployment of PICCOLO system-based vehicle applications
- Improve development productivity and shorten development cycles
- Automate quality assurance and verification
- Enhance security and standardize deployment processes

### 1.2 Overall Process Flow

```
+------------+    +------------+    +------------+    +------------+    +------------+
|            |    |            |    |            |    |            |    |            |
| Development+--->+   Build    +--->+ YAML       +--->+Deployment &+--->+  Release   |
|   Stage    |    |   Stage    |    | Generation |    |Verification |    |   Stage    |
|            |    |            |    |   Stage    |    |            |    |            |
+------------+    +------------+    +------------+    +------------+    +------------+
```

## 2. System Architecture

### 2.1 Components by Stage

Each stage consists of Frontend, Backend, and a common Database:

| Stage | Frontend | Backend | Key Functions |
|-----|----------|---------|----------|
| Development Stage | React, Monaco Editor | Node.js, Express | Project initialization, IDE provision, signal simulation |
| Build Stage | React, Redux | Node.js, Docker API | Platform selection, build execution, container creation |
| YAML Generation Stage | React, Monaco Editor | Node.js, YAML validation | YAML editing, template management, AI generation |
| Deployment Stage | React, Three.js | Spring Boot, K8s API | System mapping, digital twin verification |
| Release Stage | React, D3.js | Spring Boot, gRPC | Package management, notifications, secure deployment |

### 2.2 Database Configuration

| Database | Tech Stack | Key Stored Data |
|------------|----------|----------------|
| Metadata DB | PostgreSQL | Projects, models, regions, hardware versions, platform information |
| User DB | PostgreSQL | User information, permissions, authentication information |
| Artifact DB | MongoDB | Build artifacts, YAML templates, container metadata |
| Log/Monitoring DB | Elasticsearch | System logs, build logs, deployment history |

## 3. Detailed Specification by Stage

### 3.1 Development Stage

#### Development Stage Summary Table

| Item | Description |
|-----|------|
| **Purpose** | Project initialization and development environment configuration, signal simulation provision |
| **Key Inputs** | User authentication, project information, region/model/hardware information, repository information |
| **Key Tasks** | Project initialization, cloud IDE provision, signal simulation environment configuration |
| **Key Outputs** | Initialized project, development environment (IDE), signal simulation environment |
| **Passed to Next Stage** | Project ID, repository information, target environment information |
| **Stored Information** | Basic project information, target environment mapping, repository connection information, IDE workspace status |
| **Related DB Tables** | projects, project_regions, project_vehicle_models, project_hardware, project_repositories, ide_workspaces, vehicle_signals, signal_scenarios |

#### 3.1.1 Required Input Information and Files

- **User Authentication Information**: User ID, password
- **Project Basic Information**: Project name, description
- **Target Information**:
  - Region information (multiple selection possible)
  - Vehicle model information (multiple selection possible)
  - Hardware version information (multiple selection possible)
- **Hardware Detailed Information**:
  - Hardware vendor (e.g., Qualcomm, NVIDIA)
  - Hardware model (e.g., Snapdragon 8155, Xavier)
  - Hardware specifications (CPU, memory, GPU, etc.)
- **Source Code Repository Information**:
  - Repository URL (multiple possible)
  - Access tokens/certificates
  - Branch information
- **Development Environment Requirements**:
  - IDE settings
  - Development tool versions
  - Plugin requirements

#### 3.1.2 Stage Tasks

1. **Project Initialization**:
   - Create project metadata
   - Map regions, models, hardware
   - Set permissions

2. **Development Environment Configuration**:
   - Provision cloud IDE
   - Connect source code repositories
   - Install and configure development tools

3. **Signal Simulation Environment Configuration**:
   - Define vehicle signals
   - Create scenarios
   - Run and monitor simulations

#### 3.1.3 Outputs

- **Initialized Project**:
  - Project ID and metadata
  - Permission and team settings
  - Target environment settings

- **Development Environment**:
  - Cloud IDE URL
  - Connected repositories
  - Development tool configuration

- **Signal Simulation Environment**:
  - Defined vehicle signals
  - Created scenarios
  - Simulation sessions

#### 3.1.4 Information Passed to Next Stage

- **Project ID**: Used for project identification in the build stage
- **Repository Information**: Used for source code checkout
- **Target Environment Information**: Used for build platform selection

#### 3.1.5 Stored Information

**DB Tables**: `projects`, `project_regions`, `project_vehicle_models`, `project_hardware`, `project_repositories`, `ide_workspaces`, `vehicle_signals`, `signal_scenarios`

- Basic project information
- Target environment mapping (region, model, hardware)
- Repository connection information
- IDE workspace status
- Defined signals and scenarios

### 3.2 Build Stage

#### Build Stage Summary Table

| Item | Description |
|-----|------|
| **Purpose** | Source code building, testing, container image creation and validation |
| **Key Inputs** | Project ID, build platform information, build configuration, source code information, container configuration |
| **Key Tasks** | Build environment preparation, code compilation, test execution, container image creation, quality validation |
| **Key Outputs** | Build results (logs, test results), container images, build metadata |
| **Passed to Next Stage** | Container image information, build ID, quality analysis results |
| **Stored Information** | Build metadata, build configuration, container image information, build logs, quality analysis results |
| **Related DB Tables** | builds, container_images, build_logs |

#### 3.2.1 Required Input Information and Files

- **Project ID**: Project identifier created in the development stage
- **Build Platform Selection**:
  - Target architecture (ARM64, x86_64, etc.)
  - Target OS (QNX, AGL, Android Automotive, Linux)
  - Compiler options
- **Build Configuration**:
  - Dependency information
  - Build flags
  - Test settings
- **Source Code Information**:
  - Branch or tag
  - Commit hash (optional)
- **Container Configuration**:
  - Base image
  - Port settings
  - Volume settings
  - Registry information

#### 3.2.2 Stage Tasks

1. **Build Environment Preparation**:
   - Source code checkout
   - Build environment provisioning
   - Dependency installation

2. **Build Execution**:
   - Code compilation
   - Test execution
   - Static analysis execution

3. **Container Image Creation**:
   - Dockerfile creation/usage
   - Image building
   - Image tagging

4. **Quality Validation**:
   - Vulnerability scanning
   - Code quality inspection
   - Compliance verification

5. **Artifact Upload**:
   - Container image registry upload
   - Build log storage
   - Metadata creation

#### 3.2.3 Outputs

- **Build Results**:
  - Build status (success/failure)
  - Build logs
  - Test results
  - Quality analysis reports

- **Container Image**:
  - Image URL
  - Tags and digests
  - Size and layer information
  - Vulnerability scan results

- **Build Metadata**:
  - Build ID
  - Timestamp
  - Environment information
  - Commit information

#### 3.2.4 Information Passed to Next Stage

- **Container Image Information**: Image information to be referenced in the YAML generation stage
- **Build ID**: For build tracking and reference
- **Quality Analysis Results**: Constraints to be referenced in YAML generation

#### 3.2.5 Stored Information

**DB Tables**: `builds`, `container_images`, `build_logs`

- Build metadata (ID, status, time, user)
- Build configuration (platform, options)
- Container image information (registry, tag, digest)
- Build logs and results
- Quality analysis results

### 3.3 YAML Generation Stage

#### YAML Generation Stage Summary Table

| Item | Description |
|-----|------|
| **Purpose** | YAML configuration file generation, validation, and management for PICCOLO system |
| **Key Inputs** | Project ID, container image information, YAML type, resource requirements, specialized manager information |
| **Key Tasks** | YAML editing environment provision, YAML validation, AI-assisted YAML generation, template management |
| **Key Outputs** | Validated YAML files, validation results, saved templates |
| **Passed to Next Stage** | YAML file ID, container image reference, validation status |
| **Stored Information** | YAML file contents, metadata, template information, validation results |
| **Related DB Tables** | yaml_templates, yaml_files |

#### 3.3.1 Required Input Information and Files

- **Project ID**: Project identifier
- **Container Image Information**: Image reference created in the build stage
- **YAML Type Selection**:
  - Model, Scenario, Package, etc.
  - Specialized managers (DeviceManager, AudioManager, etc.)
- **Resource Requirements**:
  - CPU, memory requirements
  - Storage requirements
  - Network requirements
- **Specialized Manager Connection Information**:
  - Devices requiring access
  - Audio streams
  - Windows/displays
  - Network configurations
  - Storage volumes
  - Cloud services

#### 3.3.2 Stage Tasks

1. **YAML Editing Environment Provision**:
   - Direct editing mode (Monaco Editor)
   - Guided mode (wizard interface)
   - Template-based starting points

2. **YAML Validation**:
   - Syntax validation
   - Schema validity check
   - Resource constraint validation
   - Reference integrity check

3. **AI-assisted YAML Generation** (future version):
   - Natural language requirement processing
   - Context-aware suggestions
   - Template-based learning

4. **Template Management**:
   - Basic template provision
   - User-defined template storage
   - Template version management

#### 3.3.3 Outputs

- **YAML Files**:
  - Validated YAML content
  - File type (Model, Scenario, etc.)
  - Version information

- **Validation Results**:
  - Validity status
  - Warning and error list
  - Improvement suggestions

- **Templates** (optional):
  - Saved user templates
  - Template metadata

#### 3.3.4 Information Passed to Next Stage

- **YAML File ID**: YAML file reference to be used in the deployment stage
- **Container Image Reference**: Image information to be used for deployment
- **Validation Status**: Validity status of the YAML file

#### 3.3.5 Stored Information

**DB Tables**: `yaml_templates`, `yaml_files`

- YAML file contents
- YAML file metadata (ID, name, description, type)
- Template information (if used)
- Validation results and status
- Linked container image references

### 3.4 Deployment and Verification Stage

#### Deployment and Verification Stage Summary Table

| Item | Description |
|-----|------|
| **Purpose** | YAML-based application deployment, verification and testing in digital twin |
| **Key Inputs** | YAML file ID, deployment target information, system mapping information, test configuration |
| **Key Tasks** | System configuration mapping, digital twin deployment, verification test execution, result analysis |
| **Key Outputs** | Deployment results, verification test results, deployment reports |
| **Passed to Next Stage** | Deployment ID, verification result summary, deployment configuration |
| **Stored Information** | Deployment metadata, resource allocation information, test results, performance metrics |
| **Related DB Tables** | deployments, digital_twin_tests |

#### 3.4.1 Required Input Information and Files

- **YAML File ID**: File reference created in the YAML generation stage
- **Deployment Target Information**:
  - Deployment environment (digital twin, test environment)
  - Target system configuration
- **System Mapping Information**:
  - Computing node allocation
  - Network interface mapping
  - Storage volume mapping
  - Specialized resource connection
- **Test Configuration**:
  - Test scenarios
  - Test duration
  - Performance metric thresholds

#### 3.4.2 Stage Tasks

1. **System Configuration Mapping**:
   - Visual system configuration tool provision
   - Resource allocation and mapping
   - Configuration validation

2. **Digital Twin Deployment**:
   - Virtual environment preparation
   - YAML-based deployment execution
   - Deployment status monitoring

3. **Verification Test Execution**:
   - Functional tests
   - Performance tests
   - Stability tests
   - Integration tests

4. **Result Analysis and Reporting**:
   - Test result collection
   - Performance metric analysis
   - Issue identification and classification
   - Report generation

#### 3.4.3 Outputs

- **Deployment Results**:
  - Deployment status (success/failure)
  - Deployment logs
  - Resource allocation information

- **Verification Test Results**:
  - Test pass/fail status
  - Performance metrics
  - Issue list
  - Resource usage statistics

- **Deployment Report**:
  - Comprehensive results
  - Issues and improvements
  - Recommended actions

#### 3.4.4 Information Passed to Next Stage

- **Deployment ID**: Deployment identifier to be referenced in the release stage
- **Verification Result Summary**: Information to be used for release decision
- **Deployment Configuration**: Configuration information to be used for actual vehicle deployment

#### 3.4.5 Stored Information

**DB Tables**: `deployments`, `digital_twin_tests`

- Deployment metadata (ID, status, time, target)
- Resource allocation information
- Deployment logs and results
- Test execution results
- Performance metric data
- Issues and recommendations

### 3.5 Release Stage

#### Release Stage Summary Table

| Item | Description |
|-----|------|
| **Purpose** | Validated package creation, signing and encryption, vehicle deployment notification |
| **Key Inputs** | Deployment ID, YAML file ID, container image ID, release information, notification settings, signing/encryption settings |
| **Key Tasks** | Package creation, validation and approval, signing and encryption, notification dispatch, deployment monitoring |
| **Key Outputs** | Release package, notification results, deployment statistics |
| **Passed to Next Stage** | Package download information, authentication tokens, package metadata |
| **Stored Information** | Package metadata, release information, notification history, vehicle authentication information, encryption key information |
| **Related DB Tables** | packages, notifications, vehicle_authentication, encryption_keys |

#### 3.5.1 Required Input Information and Files

- **Deployment ID**: Validated deployment reference
- **YAML File ID**: Validated YAML file reference
- **Container Image ID**: Validated container image reference
- **Release Information**:
  - Package name
  - Version number
  - Release notes
- **Notification Settings**:
  - Target vehicle/system list
  - Notification priority
  - Notification message
- **Signing and Encryption Settings**:
  - Signing certificate
  - Encryption policy

#### 3.5.2 Stage Tasks

1. **Package Creation**:
   - YAML and container image packaging
   - Metadata creation
   - Version tagging

2. **Package Validation and Approval**:
   - Final validation execution
   - Approval workflow execution
   - Release permission verification

3. **Package Signing and Encryption**:
   - Package integrity signing
   - Vehicle-specific encryption key generation
   - Secure package creation

4. **Deployment Notification Dispatch**:
   - Target PICCOLO API server identification
   - Notification message creation
   - Notification dispatch and tracking

5. **Deployment Monitoring and Management**:
   - Deployment status tracking
   - Download statistics collection
   - Deployment issue response

#### 3.5.3 Outputs

- **Release Package**:
  - Package ID and metadata
  - YAML and container image references
  - Signing and encryption information

- **Notification Results**:
  - Notification status (sent/failed)
  - Receipt confirmation status
  - Dispatch time and target

- **Deployment Statistics**:
  - Download counts
  - Installation success rate
  - Error reports

#### 3.5.4 Information Passed to Next Stage

- **Package Download Information**: Information needed for vehicle systems to download the package
- **Authentication Tokens**: Used for vehicle authentication and package access
- **Package Metadata**: Used by vehicle systems to determine package suitability

#### 3.5.5 Stored Information

**DB Tables**: `packages`, `notifications`, `vehicle_authentication`, `encryption_keys`

- Package metadata (ID, name, version, status)
- Release information (release notes, publication date)
- Notification history (target, status, time)
- Vehicle authentication information
- Encryption key information (ID, expiration date)

## 4. Vehicle Deployment Process

### 4.1 Vehicle Authentication and Package Request

1. **Authentication Request**:
   - Vehicle VIN and certificate submission
   - Hardware status information provision
   - Supported encryption methods specification

2. **Authentication Confirmation**:
   - Certificate validation
   - Vehicle permission verification
   - Temporary token issuance

3. **Package Request**:
   - Token-based authentication
   - Package ID specification
   - Supported encryption level provision

4. **Package Provision**:
   - Dynamic encryption key generation
   - Encrypted YAML and key transmission
   - Container image download URL provision

### 4.2 Package Deployment Flow

```
+---------------+                          +----------------+
| Vehicle PICCOLO|                         | Cloud CI/CD    |
| API Server    |                          | Platform       |
+-------+-------+                          +--------+-------+
        |                                          |
        | 1. Authentication Request (VIN, Cert)    |
        +-------------------------------------------->
        |                                          |
        | 2. Authentication & Temporary Token      |
        <--------------------------------------------+
        |                                          |
        | 3. Package Request (Token, Encryption)   |
        +-------------------------------------------->
        |                                          |
        | 4. Dynamic Key Generation                |
        |                                          |
        | 5. Encrypted YAML & Key Delivery         |
        <--------------------------------------------+
        |                                          |
        | 6. Container Image Download URL Request  |
        +-------------------------------------------->
        |                                          |
        | 7. Signed Download URL Provision         |
        <--------------------------------------------+
        |                                          |
        | 8. Image Download & Signature Validation |
        |                                          |
        | 9. Deployment Completion Notification    |
        +-------------------------------------------->
        |                                          |
```

## 5. Database Schema Summary

### 5.1 Key Tables and Relationships

- **Project Related**: `projects`, `project_regions`, `project_vehicle_models`, `project_hardware`, `project_repositories`, `project_team_members`
- **Hardware Related**: `hardware_vendors`, `hardware_models`, `hardware`
- **Development Related**: `ide_workspaces`, `vehicle_signals`, `signal_scenarios`
- **Build Related**: `build_platforms`, `builds`, `container_images`
- **YAML Related**: `yaml_templates`, `yaml_files`
- **Deployment Related**: `deployments`, `digital_twin_tests`
- **Release Related**: `packages`, `notifications`, `vehicle_authentication`, `encryption_keys`
- **Audit and Security**: `audit_logs`, `users`

### 5.2 Important Data Flow

1. Project creation → Project ID generation → Referenced in all stages
2. Build → Container image creation → Referenced in YAML files
3. YAML file creation → Used in deployment → Included in packages
4. Deployment and verification → Verification results → Used for release decisions
5. Package creation → Notification dispatch → Vehicle download and installation

## 6. Cloud Implementation Options

### 6.1 AWS-based Implementation

| Component | AWS Service | Purpose |
|----------|-----------|------|
| Frontend | S3 + CloudFront | Static web hosting |
| Backend API | ECS/EKS + API Gateway | API services |
| Database | RDS (PostgreSQL) + DynamoDB | Metadata and artifact storage |
| Container Registry | ECR | Container image storage |
| CI/CD Pipeline | CodePipeline + CodeBuild | Build automation |
| Development Environment | EC2 + AWS Cloud9 | Cloud IDE |
| Digital Twin | IoT Core + SageMaker | Simulation environment |
| Authentication | Cognito + IAM | User and vehicle authentication |
| Security | KMS + Certificate Manager | Encryption and certificate management |
| Monitoring | CloudWatch + X-Ray | System monitoring |

### 6.2 GCP-based Implementation

| Component | GCP Service | Purpose |
|----------|-----------|------|
| Frontend | Cloud Storage + CDN | Static web hosting |
| Backend API | GKE + Cloud Endpoints | API services |
| Database | Cloud SQL + Firestore | Metadata and artifact storage |
| Container Registry | Container Registry | Container image storage |
| CI/CD Pipeline | Cloud Build + Cloud Deploy | Build automation |
| Development Environment | Compute Engine + Cloud Shell | Cloud IDE |
| Digital Twin | IoT Core + AI Platform | Simulation environment |
| Authentication | Identity Platform + IAM | User and vehicle authentication |
| Security | Cloud KMS + Certificate Authority Service | Encryption and certificate management |
| Monitoring | Cloud Monitoring + Cloud Trace | System monitoring |

## 7. Conclusion

The PICCOLO cloud CI/CD platform is an integrated solution supporting the entire lifecycle of vehicle software development from development to deployment. Each stage has clear inputs and outputs and is connected to other stages through standardized interfaces.

This platform improves developer productivity, ensures software quality, and provides a secure deployment process. It is implemented on a cloud basis to ensure scalability and availability, and all activities are auditable and traceable.

Future developments will include AI-based YAML generation, advanced analytics features, and edge computing integration.

## Appendix

### A. Terminology

| Term | Definition |
|-----|------|
| PICCOLO | Framework for in-vehicle software systems |
| YAML | Yet Another Markup Language, a configuration file format |
| CI/CD | Continuous Integration/Continuous Deployment |
| Digital Twin | Virtual replica of an actual system |
| Region | Area or country where vehicles are sold |
| HPC | High Performance Computing, high-performance computing system in vehicles |
| Specialized Manager | Component that manages specific resources in the PICCOLO system |

### B. Document History

| Version | Date | Description | Author |
|-----|------|------|-------|
| 0.1 | 2025-07-10 | Draft creation | joshua-jung_LGESDV |
| 0.5 | 2025-07-14 | Major content addition | joshua-jung_LGESDV |
| 0.9 | 2025-07-16 | Review and feedback incorporation | joshua-jung_LGESDV |
| 1.0 | 2025-07-17 | Final approval | joshua-jung_LGESDV |
| 1.1 | 2025-07-17 | Added stage summary tables | joshua-jung_LGESDV |
