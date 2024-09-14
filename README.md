# KML (Kodekloud Managed Labs) Detailed Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [System Architecture](#system-architecture)
3. [Account Lifecycle](#account-lifecycle)
4. [Key Components](#key-components)
5. [Workflow](#workflow)
6. [Security and Compliance](#security-and-compliance)
7. [Cleanup Process](#cleanup-process)
8. [Error Handling and Failsafes](#error-handling-and-failsafes)
9. [Integration Points](#integration-points)
10. [Monitoring and Logging](#monitoring-and-logging)

## 1. Introduction

KML (Kodekloud Managed Labs) is an in-house AWS (Amazon Web Services) Sandbox management system designed for educational purposes. It provides temporary, controlled AWS environments for students to practice and learn cloud computing skills in a safe, isolated setting. The system manages the lifecycle of AWS accounts, from provisioning to cleanup, ensuring efficient use of resources and maintaining security and compliance.

## 2. System Architecture

KML is built on a multi-tiered architecture:

1. **AWS Organization Layer**: 
   - Multiple AWS Organizations are used to manage accounts.
   - Each organization has its own management account.
   - Organizations are divided into Organizational Units (OUs) for different purposes (e.g., Production, Development, Testing).

2. **Account Management Layer**:
   - Handles the provisioning, assignment, and cleanup of AWS accounts.
   - Interacts with AWS Organizations API to manage accounts.

3. **User Management Layer**:
   - Creates and manages temporary IAM users for students.
   - Applies appropriate permissions based on lab requirements.

4. **Lab Type Management**:
   - Defines different lab types with specific configurations and permissions.
   - Manages policies for each lab type.

5. **Cleanup and Monitoring Layer**:
   - Continuously monitors account usage and initiates cleanup processes.
   - Ensures resources are properly terminated after lab sessions.

6. **Database Layer**:
   - Uses MongoDB to store account metadata, status, and usage history.

7. **Caching Layer**:
   - Utilizes Redis for caching frequently accessed data and managing session information.

## 3. Account Lifecycle

The lifecycle of an AWS account in KML goes through several states:

1. `account_loaded`: Initial state when an account is synced from AWS Organization.
2. `user_creating`: The system is creating a user for the account.
3. `user_created`: A user has been successfully created for the account.
4. `account_assigning`: The system is assigning the account to a student for a lab session.
5. `in_use`: The account is currently being used by a student for a lab session.
6. `account_cleanup`: The lab session has ended, and the system is cleaning up the account.
7. `user_deleted`: The user has been deleted as part of the cleanup process.
8. `account_assigning_failed`: There was an error during the account assignment process.
9. `failed`: An unexpected error occurred while the account was in use.
10. `draining_org_unit`: The account has been marked for removal from the system.

## 4. Key Components

1. **AWSPlatformAccountManager**: 
   - Manages the overall lifecycle of AWS accounts.
   - Handles account synchronization, user creation, and account maintenance.

2. **AWSPlatformAccountAssigner**:
   - Assigns accounts to students based on lab type and duration.
   - Applies appropriate policies and configurations.

3. **AWSPlatformAccountCleaner**:
   - Responsible for cleaning up accounts after lab sessions.
   - Analyzes CloudTrail events to determine which resources to clean.

4. **AWSPlatformAccountReadinessChecker**:
   - Ensures accounts are ready for use before assignment.

5. **Failsafe Mechanism**:
   - Monitors for repeated failures and can deactivate lab types if issues persist.

6. **Policy Management**:
   - Manages IAM policies for different lab types.
   - Applies time-bound permissions to control access.

## 5. Workflow

1. **Account Synchronization**:
   - KML regularly syncs with AWS Organizations to maintain an up-to-date pool of available accounts.

2. **User Request**:
   - A student requests a lab environment of a specific type and duration.

3. **Account Assignment**:
   - KML selects an available account and creates a temporary IAM user.
   - Appropriate policies are applied based on the lab type.

4. **Lab Session**:
   - The student uses the provided credentials to access the AWS environment.
   - KML monitors the usage and enforces time limits.

5. **Cleanup**:
   - After the lab session ends, KML initiates the cleanup process.
   - CloudTrail logs are analyzed to determine which resources need to be removed.
   - A series of cleanup scripts are executed to remove all created resources.

6. **Reset**:
   - The account is reset to its initial state and marked as available for future use.

## 6. Security and Compliance

1. **Time-bound Access**: All permissions are time-limited to the duration of the lab session.
2. **Least Privilege**: Policies are crafted to provide only the necessary permissions for each lab type.
3. **Isolation**: Each student gets a separate AWS account, ensuring complete isolation of resources.
4. **Automatic Cleanup**: Ensures that no sensitive data or resources persist after a lab session.
5. **Service Control Policies (SCPs)**: Applied at the organization level to enforce global restrictions.

## 7. Cleanup Process

The cleanup process is a critical component of KML:

1. **Event Analysis**: CloudTrail events are analyzed to determine which AWS services were used.
2. **Resource Identification**: Based on the events, KML identifies all resources that need to be cleaned up.
3. **Cleanup Execution**: A series of cleanup functions are executed, each targeting specific AWS services.
4. **Verification**: KML verifies that all resources have been properly removed.
5. **Account Reset**: The account is reset to its initial state, ready for the next use.

## 8. Error Handling and Failsafes

1. **Account Failure Tracking**: If an account encounters repeated failures, it's marked for investigation.
2. **Lab Type Deactivation**: If a specific lab type causes multiple failures, it can be automatically deactivated.
3. **Logging and Monitoring**: Extensive logging is implemented to track errors and system behavior.

## 9. Integration Points

1. **AWS Organizations API**: For account management and synchronization.
2. **AWS IAM**: For user and policy management.
3. **AWS CloudTrail**: For monitoring account activity and resource creation.
4. **MongoDB**: For storing account metadata and usage history.
5. **Redis**: For caching and session management.

## 10. Monitoring and Logging

1. **Account Usage Monitoring**: Tracks the number of accounts in various states.
2. **Error Logging**: Detailed error logging for troubleshooting and system improvement.
3. **Activity Logging**: Logs all major activities like account assignments, cleanups, and policy applications.
4. **Performance Metrics**: Tracks system performance, including account provisioning and cleanup times.
