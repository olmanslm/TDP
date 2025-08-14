# AEM Core Components Installation Troubleshooting - Technical Documentation

## Problem Summary

The AEM Core Components were not being installed in the local AEM environment despite being configured in the project's Maven dependencies. The Core Components bundle (`com.adobe.cq.wcm.core.components.core`) was missing from the OSGi console, preventing the use of Core Components in page authoring.

## Root Cause Analysis

### Initial Configuration Issues Identified

1. **Missing Core Bundle in Deployment Package**
   - The `all/pom.xml` only included Core Components example packages
   - The actual Core Components bundle (`core.wcm.components.core`) was missing from the embedded dependencies
   - Only example packages were being deployed:
     - `core.wcm.components.examples.ui.apps`
     - `core.wcm.components.examples.ui.content` 
     - `core.wcm.components.examples.ui.config`

2. **Dependency Resolution Problem**
   - Core Components bundle was referenced without explicit version in `all/pom.xml`
   - Maven could not resolve the dependency properly during the packaging phase

### Technical Analysis Process

1. **Build Log Investigation**
   ```
   Package installation log showed:
   ✅ core.wcm.components.examples.ui.config-2.28.0.zip
   ✅ core.wcm.components.examples.ui.apps-2.28.0.zip  
   ✅ core.wcm.components.examples.ui.content-2.28.0.zip
   ❌ Missing: core.wcm.components.core-2.x.x.jar
   ```

2. **Package Content Verification**
   ```bash
   # Verified built package contents
   find ./all/target/vault-work -name "*core.wcm.components*" -type f
   # Result: No core bundle found in package
   ```

3. **Dependency Tree Analysis**
   - Confirmed Core Components version was properly defined in parent POM
   - Identified version resolution issue in `all` module

## Solution Implementation

### Step 1: Add Core Components Bundle to Embedded Dependencies

**File**: `all/pom.xml`

**Added to `<embeddeds>` section:**
```xml
<embedded>
    <groupId>com.adobe.cq</groupId>
    <artifactId>core.wcm.components.core</artifactId>
    <target>/apps/criticalmass-vendor-packages/application/install</target>
</embedded>
```

**Installation Path**: `/apps/criticalmass-vendor-packages/application/install`

### Step 2: Add Core Components Bundle to Dependencies

**File**: `all/pom.xml`

**Added to `<dependencies>` section:**
```xml
<dependency>
    <groupId>com.adobe.cq</groupId>
    <artifactId>core.wcm.components.core</artifactId>
    <version>${core.wcm.components.version}</version>
</dependency>
```

**Critical Fix**: The explicit version reference `${core.wcm.components.version}` was essential for proper Maven dependency resolution.

### Step 3: Version Management

**Parent POM Configuration** (already correct):
```xml
<properties>
    <core.wcm.components.version>2.28.0</core.wcm.components.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.adobe.cq</groupId>
            <artifactId>core.wcm.components.core</artifactId>
            <version>${core.wcm.components.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## Solution Architecture

### Before (Problematic Configuration)

```
Project Structure:
├── Parent POM
│   ├── ✅ Core Components version defined (2.28.0)
│   └── ✅ Dependency management configured
├── Core Module
│   └── ✅ Core Components (test scope only)
└── All Module (Deployment Package)
    ├── ❌ Missing core bundle embedding
    ├── ❌ Missing core bundle dependency
    └── ✅ Only example packages included
```

### After (Working Configuration)

```
Project Structure:
├── Parent POM
│   ├── ✅ Core Components version defined (2.28.0→2.29.0)
│   └── ✅ Dependency management configured
├── Core Module
│   └── ✅ Core Components (test scope only)
└── All Module (Deployment Package)
    ├── ✅ Core bundle embedded with explicit version
    ├── ✅ Core bundle dependency declared
    ├── ✅ Example packages included
    └── ✅ Proper installation paths configured
```

## Verification Process

### Build Output Confirmation

```
[INFO] Embedding com.adobe.cq:core.wcm.components.core:jar:2.29.0 
-> jcr_root/apps/criticalmass-vendor-packages/application/install/core.wcm.components.core-2.29.0.jar

[INFO] Processing bundle core.wcm.components.core-2.29.0.jar...
```

### Deployment Verification

1. **OSGi Console Check**: `http://localhost:4502/system/console/bundles`
   - Bundle: "Adobe Experience Manager Core WCM Components Bundle"
   - Status: Active
   - Version: 2.29.0

2. **Package Manager Verification**: Package successfully installed with all components

## Key Learnings & Best Practices

### 1. AEM Package Deployment Requirements

- **Example packages alone are insufficient** - they contain sample content but not the functional code
- **Core bundle is mandatory** - contains the actual component implementations (Java classes, models, services)
- **All packages must be embedded** in the deployment module for installation

### 2. Maven Dependency Management

- **Explicit versioning required** in deployment modules
- **Property-based version management** ensures consistency across modules
- **Dependency resolution order** matters in multi-module projects

### 3. AEM Installation Paths

- **Application bundles**: `/apps/[vendor]-packages/application/install/`
- **Content packages**: `/apps/[vendor]-packages/content/install/`
- **Vendor separation**: Keep third-party packages in separate vendor directories

## Recommended Team Procedures

### 1. Pre-Deployment Checklist

```bash
# Verify Core Components in package
find ./all/target/vault-work -name "*core.wcm.components*" -type f

# Check dependency resolution
mvn dependency:tree -pl all | grep core.wcm.components

# Validate build output
mvn clean install -pl all | grep "Embedding.*core.wcm.components.core"
```

### 2. Post-Deployment Verification

- [ ] OSGi Console: Core Components bundle active
- [ ] Package Manager: All packages installed successfully
- [ ] Component Browser: Core Components available in authoring
- [ ] Sample Page: Core Components functional

### 3. Version Update Process

1. Update `core.wcm.components.version` in parent POM
2. Rebuild and test locally
3. Verify in staging environment
4. Document changes for team

## Tools & Commands Used

```bash
# Build and deploy specific module
mvn clean install -pl all -PautoInstallSinglePackage

# Full project build and deploy
mvn clean install -PautoInstallSinglePackage

# Dependency analysis
mvn dependency:tree -pl all

# Package content verification
find ./all/target/vault-work -name "*.jar" -o -name "*.zip"
```

## Final Configuration State

**Version Upgraded**: 2.28.0 → 2.29.0 (latest available)

**Deployed Components**:
- ✅ `core.wcm.components.core-2.29.0.jar` (Core bundle)
- ✅ `core.wcm.components.examples.ui.apps-2.29.0.zip`
- ✅ `core.wcm.components.examples.ui.config-2.29.0.zip`
- ✅ `core.wcm.components.examples.ui.content-2.29.0.zip`

## Code Changes Made

### Modified: `all/pom.xml`

#### Embedded Dependencies Section
```xml
<!-- Added Core Components bundle embedding -->
<embedded>
    <groupId>com.adobe.cq</groupId>
    <artifactId>core.wcm.components.core</artifactId>
    <target>/apps/criticalmass-vendor-packages/application/install</target>
</embedded>
```

#### Dependencies Section
```xml
<!-- Added Core Components bundle dependency with explicit version -->
<dependency>
    <groupId>com.adobe.cq</groupId>
    <artifactId>core.wcm.components.core</artifactId>
    <version>${core.wcm.components.version}</version>
</dependency>
```

## Troubleshooting Tips for Future Issues

### If Core Components are Missing:

1. **Check OSGi Console**: Look for `com.adobe.cq.wcm.core.components.core` bundle
2. **Verify Package Contents**: Use `find` command to check embedded artifacts
3. **Check Build Logs**: Look for "Embedding" messages during Maven build
4. **Validate Dependencies**: Use `mvn dependency:tree` to verify resolution

### Common Mistakes to Avoid:

- ❌ Only including example packages without the core bundle
- ❌ Missing explicit version in deployment module dependencies
- ❌ Wrong installation paths for bundles vs packages
- ❌ Forgetting to update version properties when upgrading

## Follow-up Issue: Page Component Inheritance Problem

### Problem Description

After successfully installing the Core Components bundle, a new issue emerged during page rendering:

```
ERROR: No renderer for extension html, cannot render resource 
JcrNodeResource, type=criticalmass/components/page, superType=null
```

**Key Symptoms**:
- Core Components bundle was active in OSGi console
- Custom page component had correct `sling:resourceSuperType="core/wcm/components/page/v3/page"`
- Sling was showing `superType=null` instead of recognizing the inheritance chain

### Advanced Troubleshooting Process

#### Step 1: Component Accessibility Verification

**Problem Investigation**:
```bash
# Verified custom component was deployed correctly
curl -s -u admin:admin "http://localhost:4502/apps/criticalmass/components/page.json"
# Result: ✅ Component found with correct sling:resourceSuperType

# Checked if Core Components page was accessible
curl -s -u admin:admin "http://localhost:4502/apps/core/wcm/components/page/v3/page.json"
# Result: ❌ "File not found" - Core Components not in repository!
```

**Root Cause Discovered**: The Core Components **bundle** was installed (OSGi), but the **component definitions** were not deployed to the repository (`/apps/core/wcm/components/`).

#### Step 2: Package Analysis

**Understanding the Architecture**:
- `core.wcm.components.core` = OSGi bundle (Java classes, services, models)
- `core.wcm.components.content` = Content package (component definitions, scripts, dialogs)
- `core.wcm.components.examples.*` = Sample implementations

**The Missing Piece**: We had the code (bundle) but not the component definitions (content package).

#### Step 3: Bundle vs Content Package Distinction

| Package Type | Contains | Purpose | Installation Path |
|--------------|----------|---------|-------------------|
| `core.wcm.components.core` | Java classes, OSGi services | Business logic | OSGi runtime |
| `core.wcm.components.content` | Component definitions, HTL scripts | Repository structure | `/apps/core/wcm/components/` |
| `core.wcm.components.examples.*` | Sample content | Demonstration | `/apps/core/wcm/components/` |

### Solution Implementation - Phase 2

#### Step 1: Add Missing Content Package

**File**: `all/pom.xml`

**Added to embedded dependencies**:
```xml
<embedded>
    <groupId>com.adobe.cq</groupId>
    <artifactId>core.wcm.components.content</artifactId>
    <type>zip</type>
    <target>/apps/criticalmass-vendor-packages/application/install</target>
</embedded>
```

**Added to dependencies**:
```xml
<dependency>
    <groupId>com.adobe.cq</groupId>
    <artifactId>core.wcm.components.content</artifactId>
    <type>zip</type>
    <version>${core.wcm.components.version}</version>
</dependency>
```

#### Step 2: Complete Build and Deployment

```bash
mvn clean install -PautoInstallSinglePackage
```

#### Step 3: Verification of Resolution

**Component Accessibility Check**:
```bash
curl -s -u admin:admin "http://localhost:4502/apps/core/wcm/components/page/v3/page.json"
```

**Successful Result**:
```json
{
  "jcr:primaryType": "cq:Component",
  "jcr:title": "Page (v3)",
  "sling:resourceSuperType": "wcm/foundation/components/basicpage/v1/basicpage",
  "componentGroup": ".core-wcm"
}
```

**Page Rendering Test**:
```bash
curl -s -u admin:admin "http://localhost:4502/content/criticalmass/us/en.html"
```

**Result**: ✅ Page renders successfully without `superType=null` error.

### Complete Architecture Understanding

#### Final Working Configuration

```
AEM Core Components Complete Setup:
├── OSGi Bundle (core.wcm.components.core)
│   ├── Java Models (PageImpl.class, etc.)
│   ├── OSGi Services
│   └── Sling Adapters
├── Content Package (core.wcm.components.content)
│   ├── Component Definitions (/apps/core/wcm/components/)
│   ├── HTL Scripts (page.html, body.html, etc.)
│   ├── Dialog Definitions
│   └── Component Metadata
└── Example Packages (optional)
    ├── Sample Content
    └── Demo Implementations
```

#### Inheritance Chain Resolution

```
Custom Page Component Inheritance:
criticalmass/components/page
├── sling:resourceSuperType="core/wcm/components/page/v3/page"
└── Resolves to: /apps/core/wcm/components/page/v3/page
    ├── sling:resourceSuperType="wcm/foundation/components/basicpage/v1/basicpage"
    └── Renders with: page.html script
```

### Key Diagnostic Commands

```bash
# Verify OSGi bundle status
curl -s -u admin:admin "http://localhost:4502/system/console/bundles" | grep -i "core.*wcm"

# Check component accessibility
curl -s -u admin:admin "http://localhost:4502/apps/core/wcm/components/page/v3/page.json"

# Test custom component deployment
curl -s -u admin:admin "http://localhost:4502/apps/criticalmass/components/page.json"

# Verify inheritance chain
curl -s -u admin:admin "http://localhost:4502/content/criticalmass/us/en.html"
```

### Advanced Learning Points

#### 1. AEM Component Architecture

**Critical Understanding**: Core Components require **both**:
- OSGi bundle for business logic
- Content package for repository structure

Missing either results in different failure modes:
- No bundle: `ClassNotFoundException`, service resolution errors
- No content: `superType=null`, component not found errors

#### 2. Sling Resource Resolution

**How Sling Resolves superType**:
1. Checks if resource type exists in repository
2. Reads `sling:resourceSuperType` property
3. Recursively resolves inheritance chain
4. Falls back to default renderer if chain breaks

**Failure Point**: If `/apps/core/wcm/components/page/v3/page` doesn't exist, Sling cannot resolve the superType, resulting in `superType=null`.

#### 3. Maven Dependency Types

- `jar` dependencies → OSGi bundles
- `zip` dependencies → Content packages
- Both required for complete AEM component libraries

### Final Deployment Verification

**Complete Package List**:
```
Deployed Artifacts:
├── ✅ core.wcm.components.core-2.29.0.jar (OSGi Bundle)
├── ✅ core.wcm.components.content-2.29.0.zip (Component Definitions)
├── ✅ core.wcm.components.examples.ui.apps-2.29.0.zip
├── ✅ core.wcm.components.examples.ui.config-2.29.0.zip
└── ✅ core.wcm.components.examples.ui.content-2.29.0.zip
```

**OSGi Console Status**:
- Bundle ID: 560
- Name: "Adobe Experience Manager Core WCM Components Core Bundle"
- State: Active
- Version: 2.29.0

**Repository Structure Created**:
```
/apps/core/wcm/components/
├── page/v3/page/ (✅ Now available)
├── text/v2/text/
├── title/v3/title/
└── [all other Core Components]
```

### Team Best Practices - Updated

#### 1. Complete Dependency Checklist

When adding AEM component libraries:
- [ ] OSGi bundle(s) added to embedded dependencies
- [ ] Content package(s) added to embedded dependencies  
- [ ] Both added to Maven dependencies with explicit versions
- [ ] Build includes all required artifacts
- [ ] Post-deployment verification covers both OSGi and repository

#### 2. Troubleshooting Hierarchy

**For component inheritance issues**:
1. Verify custom component deployment and configuration
2. Check superType component accessibility in repository
3. Validate OSGi bundle status for business logic
4. Test complete inheritance chain resolution

#### 3. Common Pitfalls - Extended

- ❌ Installing only OSGi bundles without content packages
- ❌ Installing only content packages without OSGi bundles
- ❌ Assuming example packages contain core functionality
- ❌ Not verifying repository structure after deployment
- ❌ Confusing bundle status with component availability

---

**Document Version**: 2.0  
**Date**: August 14, 2025  
**Author**: GitHub Copilot && Olman San Lee.
**Project**: Critical Mass AEM Multi-Site  
**Status**: ✅ Fully Resolved - Both Bundle and Content Issues
