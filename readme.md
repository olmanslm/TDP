# TDP (Technology Development Platform)

This repository contains essential tools and resources for Adobe Experience Manager (AEM) development, including the AEM runtime environment, Java Development Kit, and relevant documentation.

## Related Repository

🔗 **Project Repository:** [criticalmass-aem](https://github.com/olmanslm/criticalmass-aem)

## Directory Structure

### 📁 AEM/

Adobe Experience Manager related files and installations.

#### author/

Contains AEM author instance files for content creation and management.

##### 6.6/

- **`cq-quickstart-6.6.0.jar`** - Adobe Experience Manager 6.6 QuickStart JAR file. This is the main executable that starts the AEM author instance. Use this to launch a local AEM development environment for content authoring, component development, and testing.
- Find the file here: https://oneomnicom.sharepoint.com/:f:/r/sites/CMTechnology/Shared%20Documents/CM%20AEM/Installations/AEM-6.6?csf=1&web=1&e=oEg6Id 
- core components https://github.com/adobe/aem-core-wcm-components/releases/tag/core.wcm.components.reactor-2.28.0 

### 📁 Book/

Documentation and learning resources.

- **`Extend and Customize.pdf`** - Technical documentation focused on extending and customizing AEM functionality. This resource likely covers development patterns, best practices, and advanced customization techniques for AEM implementations.

### 📁 JDK/

Java Development Kit installation files.

- **`jdk-17.0.15_macos-aarch64_bin.dmg`** - Java Development Kit 17 installer for macOS (Apple Silicon/ARM64 architecture). JDK 17 is required to run AEM 6.6 and develop Java-based AEM components and services.

### 📁 Resources/

Development environment configuration files.

- **`.zshrc`** - Zsh shell configuration file with environment settings and aliases for AEM development workflow.

### 📁 criticalmass-aem/

The main AEM project repository containing the complete Critical Mass AEM implementation.

This directory contains the full AEM Maven project structure including:
- **Core modules** (`core/`) - Java backend logic and services
- **UI modules** (`ui.apps/`, `ui.content/`, `ui.frontend/`) - Frontend components, templates, and content
- **Configuration** (`ui.config/`) - OSGi configurations and settings
- **Tests** (`ui.tests/`, `it.tests/`) - Unit and integration tests
- **Dispatcher** (`dispatcher/`) - Apache dispatcher configuration
- **Build configuration** (`pom.xml`, `Makefile`) - Maven and build setup
- **Cloud Manager** (`.cloudmanager/`) - Adobe Cloud Manager pipeline configuration

## Getting Started

### Prerequisites

1. Install JDK 17 using the provided DMG file in the `JDK/` directory
2. Ensure you have sufficient system resources (minimum 4GB RAM recommended for AEM)

### Java Environment Management with jEnv

This project recommends using **jEnv** for managing multiple Java versions on your system.

🔗 **jEnv Official Site:** [https://www.jenv.be/](https://www.jenv.be/)

**What is jEnv?**
jEnv is a command-line tool that allows you to switch between different Java versions seamlessly. This is particularly useful when working with multiple projects that require different Java versions.

**Why use jEnv with AEM development?**
- **Version Control:** Easily switch between Java versions for different projects
- **Project-specific Java:** Set specific Java versions per project directory
- **Global and Local Settings:** Configure global defaults and project-specific overrides
- **Shell Integration:** Works seamlessly with your terminal environment

**Basic jEnv Commands:**
```bash
# List available Java versions
jenv versions

# Set global Java version
jenv global 17.0

# Set local Java version for current project
jenv local 17.0

# Add a new Java installation to jEnv
jenv add /path/to/java/home
```

**Setup for this project:**
1. Install jEnv following instructions at [jenv.be](https://www.jenv.be/)
2. Add your installed JDK 17 to jEnv
3. Set Java 17 as the local version for this project directory
4. Source the `.zshrc` file from `Resources/` for optimized environment settings

### Running AEM author Instance

1. Navigate to `AEM/author/6.6/`
2. Run the QuickStart JAR:

   ```bash
   java -jar cq-quickstart-6.6.0.jar
   ```

3. Access the AEM author interface at `http://localhost:4502`
4. Default credentials: admin/admin

### Development Resources

- Refer to `Book/Extend and Customize.pdf` for advanced development guidance
- Use the AEM author instance for component development and testing
- JDK 17 provides the Java runtime environment needed for AEM development
- The `criticalmass-aem/` directory contains the complete project source code
- Use `Resources/.zshrc` for optimized shell environment settings

## Notes

- This setup is designed for local development and testing
- AEM 6.6 requires JDK 17 for optimal performance and compatibility
- The author instance is used for content creation, component development, and administrative tasks

## System Requirements

- macOS (Apple Silicon/ARM64)
- Minimum 4GB RAM (8GB+ recommended)
- Java 17 (included in JDK folder)
- Available disk space: ~2GB for AEM installation
- \

## TDP Sharepoint link
- 