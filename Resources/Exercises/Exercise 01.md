# Mastering AEM Logging and Component Development: Hands-On Exercise

## Introduction

Adobe Experience Manager (AEM) is a powerful platform for building enterprise web applications. In this hands-on exercise, you'll learn two essential skills for AEM developers:

1. **Custom Logging in AEM**: Discover how to create and configure custom loggers, understand the Additive property, and manage log output for your services. This will help you debug, monitor, and maintain your applications more effectively.
2. **Component Development**: Build a demo component from scratch, including dialog fields, HTL templates, Sling Models, and service calls. You'll see how to connect backend logic to authoring dialogs and front-end rendering, just like in real-world AEM projects.

## 🎯 Learning Objectives
- Understand what the `Additive` property does in log configuration.
- Create a **custom logger** for a specific package in AEM.
- Verify how logs are written to both a custom file and the global `error.log`.
- Create a new AEM component that demonstrates dialog fields, templates, Sling Models, and a service call.

---
# Exercise: Custom Logging and Component Demo in AEM

Welcome! 🎉 In this exercise, you’ll learn how to:
- Create and understand custom logs in AEM OSGi (Additive property)
- Build a demo component that uses dialog fields, templates, Sling Models, and a service call

---
## Part 1: Custom Logging in AEM
### Understanding the Additive Property in AEM Logging

The `additive` property controls whether log messages from a custom logger are also forwarded to the root logger (typically `error.log`):

- **additive = true**: Messages are written to both your custom log file and the global log.
- **additive = false**: Messages are written only to your custom log file.

#### When to Use `additive = true`

Use this setting when you want log entries for a specific package or class to appear in both a dedicated log file and the main global log. This is helpful for debugging, as you get focused logs without losing visibility in the overall system logs.

#### When to Use `additive = false`

Choose this setting to fully isolate log messages in a custom file, preventing duplication in the global log. This is ideal for packages that generate a lot of log output and could clutter the main logs.

In summary, set `additive` according to your need for visibility versus isolation of log messages.

### Step 1: Open the Sling Log Console

1. Navigate to: [http://localhost:4502/system/console/slinglog](http://localhost:4502/system/console/slinglog)
2. You will see a list of current loggers and appenders.

### Step 2: Create and Configure a New Logger

1. Scroll down and click **Add new Logger**.
2. Fill in the logger configuration fields as follows:
    - **Log Level**: `DEBUG`
    - **Additive**: `true`
    - **Log File**: `logs/my-custom-service.log`
    - **Logger**: `com.criticalmass.core.services`
3. Click **Save** to create the logger.

Your custom logger is now set up to capture log messages from the `com.criticalmass.core.services` package. With `Additive = true`, these messages will appear in both your custom log file and the global `error.log`.

### Step 3: Create your own custom GreetingService (be creative)

In your AEM project, create or update the following files inside the package `com.criticalmass.core.services`:

#### GreetingService.java

```java
package com.criticalmass.core.services;

/**
 * Service interface definition.
 */
public interface GreetingService {
    String getMessage(String name, int number);
}
```

#### GreetingServiceImpl.java

```java
package com.criticalmass.core.services.impl;

import com.criticalmass.core.services.GreetingService;
import org.osgi.service.component.annotations.Component;
import lombok.extern.slf4j.Slf4j;

@Component(service = GreetingService.class)
@Slf4j
public class GreetingServiceImpl implements GreetingService {
    @Override
    public String getMessage(String name, int number) {
        log.debug("Processing started for name: {} and number: {}", name, number);
        log.info("Business logic running");
        log.error("Something went wrong!");
        return String.format("Hello %s! Your number is %d.", name, number);
    }
}
```

### Step 4: Test the Logging Behavior

1. Trigger your code (e.g., run the service or call the method).
2. Now check the logs:
   - Open `logs/my-custom-service.log` — you should see your log messages.
   - Open `logs/error.log` — you’ll also see the same messages here.

This happens because `Additive = true` propagates the log entries to the root logger.

### Step 5: Toggle Additive and Test Again

1. Go back to the Sling Log Console (`/system/console/slinglog`).
2. Edit your custom logger and set **Additive** to `false`.
3. Save the changes.
4. Trigger your code again (run the service or call the method).
5. Check the logs:
    - Open `logs/my-custom-service.log` — your log messages should still appear here.
    - Open `logs/error.log` — your log messages should NOT appear here anymore.

This confirms that with `Additive = false`, log entries are isolated to your custom log file and do not propagate to the root logger.

#### Additive = true

Logs are written to the custom file AND also propagated to the root logger (`error.log`):

```text
[MyCustomService] ---> [my-custom-service.log]
    |
    +-----------> [error.log]
```

#### Additive = false

Logs are written only to the custom file, no propagation:

```text
[MyCustomService] ---> [my-custom-service.log]
```

> **Deprecated:** Advanced Exercise: Configure via OSGi XML
>
> This method is deprecated and should not be used for new AEM projects. Use OSGi JSON configs instead.
>
> Instead of creating the logger manually in `/system/console/slinglog`, you can configure it via OSGi XML.
>
> **Example:** `org.apache.sling.commons.log.LogManager.factory.config-criticalmass.xml`
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <configuration
>     xmlns:osgi="http://www.osgi.org/xmlns/metatype/v1.1.0"
>     pid="org.apache.sling.commons.log.LogManager.factory.config">
>     <property name="org.apache.sling.commons.log.file" value="logs/my-custom-service.log"/>
>     <property name="org.apache.sling.commons.log.level" value="DEBUG"/>
>     <property name="org.apache.sling.commons.log.names" value="com.criticalmass.core.services"/>
>     <property name="org.apache.sling.commons.log.additive" value="true"/>
> </configuration>
> ```
>
> **Deploying:**
> `ui.config/src/main/content/jcr_root/apps/criticalmass/osgiconfig/config`

## 🎓 Advanced Exercise: Configure via OSGi JSON

Instead of creating the logger manually in `/system/console/slinglog`, you can configure it via OSGi JSON config file (recommended for latest AEM versions).

### Step 6: Logger only for TDP runmode
`org.apache.sling.commons.log.LogManager.factory.config~tdp.cfg.json`

```json
{
    "org.apache.sling.commons.log.names": [
        "com.criticalmass.core.services"
    ],
    "org.apache.sling.commons.log.level": "DEBUG",
    "org.apache.sling.commons.log.file": "logs/my-custom-service-tdp.log",
    "org.apache.sling.commons.log.additiv": "true"
}
```

**Deploying:**
1. Create the folder for the TDP runmode config if it does not exist:
    `ui.config/src/main/content/jcr_root/apps/criticalmass/osgiconfig/config.tdp`
2. Place your config file (e.g. `org.apache.sling.commons.log.LogManager.factory.config~tdp.cfg.json`) inside this folder.
3. run `make author-fast` in the terminal from the root folder.

---
## 📝 Step 7: Create a New Logger via OSGi JSON

1. In your AEM project, navigate to:
    `ui.config/src/main/content/jcr_root/apps/criticalmass/osgiconfig/config`
2. Create a new file named (for example):
    `org.apache.sling.commons.log.LogManager.factory.config~mycustomservice.cfg.json`
3. Add the following content to define your custom logger:
    ```json
    {
      "org.apache.sling.commons.log.names": [
         "com.criticalmass.core.services"
      ],
      "org.apache.sling.commons.log.level": "DEBUG",
      "org.apache.sling.commons.log.file": "logs/my-custom-service.log",
      "org.apache.sling.commons.log.additiv": "true"
    }
    ```
4. Build and deploy your project.
5. After deployment, check `/system/console/slinglog` to confirm your new logger appears and is active.
6. To change the behavior, set `org.apache.sling.commons.log.additiv` to `false`.
7. run `make author-fast` in the terminal from the root folder.
8. restart the AEM server for the changes to take effect.

This approach allows you to manage loggers as code and version them in your repository.

---
## Part 2: Component Demo Exercise

---

### Step 1: Create a New Component Folder
In your project, under `criticalmass-aem/ui.apps/src/main/content/jcr_root/apps/criticalmass/components/`, create a folder named after your component, e.g. `mydemo`.

---

### Step 2: Add a Component HTL File
Create `mydemo.html` in your new folder. Use the following features:
- Call a template with parameters using `data-sly-call`.
- Render dialog field values using `${properties.fieldName}`.
- Use a Sling Model (e.g. `com.criticalmass.core.models.MyDemoModel`) with `data-sly-use`.
- Display output from a service call via the Sling Model.

Example structur (Be creative):
```html
<sly data-sly-call="${greet @ name='Demo User', number=123}"></sly>
<template data-sly-template.greet="${@ name, number}">
    <h2>Hello ${name}, your number is ${number}!</h2>
</template>
<div>
    <sly data-sly-test="${properties.text}" data-sly-call="${greet @ name='Dialog User', number=properties.number}"></sly>
    <div data-sly-test="${properties.text}">
        <p>Text property:</p>
        <pre>${properties.text}</pre>
    </div>
    <sly data-sly-use.model="com.criticalmass.core.models.MyDemoModel"></sly>
    <div data-sly-test="${model.message}">
        <p>Model message:</p>
        <pre>${model.message}</pre>
        <p>Service call result:</p>
        <pre>${model.greetingService}</pre>
    </div>
</div>
```

---

### Step 3: Create a Dialog for Your Component and other files
Create the following folders inside your component directory (e.g., `criticalmass-aem/ui.apps/src/main/content/jcr_root/apps/criticalmass/components/helloworld/`):

- `_cq_dialog`
- `_cq_htmlTag`
- `_cq_editConfig`

Inside each folder, add a `.content.xml` file containing the respective configuration:

- `_cq_dialog/.content.xml` — dialog definition (fields like `text`, `number` and others)
- `_cq_htmlTag/.content.xml` — HTML tag configuration
- `_cq_editConfig/.content.xml` — edit config settings

All files inside these folders should be named `.content.xml`.

---

### Step 4: Implement a Sling Model
Create a Java class (e.g. `MyDemoModel`) in `criticalmass-aem/core/src/main/java/com/criticalmass/core/models/` that injects dialog properties and calls your service (e.g. `GreetingService`).

---

### Step 5: Demo the Component
- Add the component to a new page called exercise-01-&lt;yourName&gt;.
- Fill dialog fields and verify output.
- Ensure the service call result is displayed.
- create a content package with your testing page

---
## Final Submission: Branch, Commit, and Pull Request

To submit your completed exercise, follow these steps:

1. Ensure all your changes (Java code, OSGi JSON logger config files, and documentation) are saved in your local repository of the criticalmass-aem repo.
2. Create a folder in the root folder in your repository named after yourself `results-<yourName>`, and place all your exercise files (logs, documentation and others) inside this folder.
3. Create a new branch from `master`:
    ```sh
    git checkout master
    git pull
    git checkout -b feature/aem-custom-logger-exercise-<yourName>
    ```
4. Add and commit all your changes:
    ```sh
    git add .
    git commit -m "AEM custom logger exercise: code, config, and docs -- Author: <yourName>"
    ```
5. Push your branch to the remote repository:
    ```sh
    git push -u origin feature/aem-custom-logger-exercise
    ```
6. Open a Pull Request (PR) from your branch to `master` in our project repository [criticalmass-aem](https://github.com/olmanslm/criticalmass-aem).
7. In the PR description, briefly summarize what you implemented and tested (custom logger, code, config, and documentation).

Be sure to include your name(s) in the PR description and title so your submission can be identified and credited.

---
