# Migration Reference Guide - File-by-File Changes

> Quick reference for implementing the Akka to Pekko migration
> 
> See [MIGRATION_COMPATIBILITY_REPORT.md](./MIGRATION_COMPATIBILITY_REPORT.md) for full analysis

## Overview

This document provides specific, actionable changes needed for each file during the migration.

---

## 1. Dependency Updates (POM Files)

### 1.1 Root pom.xml

**Location:** `/pom.xml`

**Changes:**
```xml
<!-- BEFORE -->
<typesafe.akka.version>2.5.22</typesafe.akka.version>
<play2.version>2.7.2</play2.version>
<scala.version>2.12.11</scala.version>
<scala.major.version>2.12</scala.major.version>

<!-- AFTER -->
<pekko.version>1.0.3</pekko.version>
<play2.version>2.9.5</play2.version>
<scala.version>2.13.14</scala.version>
<scala.major.version>2.13</scala.major.version>
```

### 1.2 sb-actor/pom.xml

**Location:** `/sb-actor/pom.xml`

**Remove:**
```xml
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_${scala.major.version}</artifactId>
    <version>${typesafe.akka.version}</version>
</dependency>
```

**Add:**
```xml
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-actor_${scala.major.version}</artifactId>
    <version>${pekko.version}</version>
</dependency>
```

### 1.3 all-actors/pom.xml

**Location:** `/all-actors/pom.xml`

**Remove:**
```xml
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-testkit_${scala.major.version}</artifactId>
    <version>${typesafe.akka.version}</version>
    <scope>test</scope>
</dependency>
```

**Add:**
```xml
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-testkit_${scala.major.version}</artifactId>
    <version>${pekko.version}</version>
    <scope>test</scope>
</dependency>
```

### 1.4 service/pom.xml

**Location:** `/service/pom.xml`

**Update Play Dependencies:**
```xml
<!-- BEFORE -->
<dependency>
    <groupId>com.typesafe.play</groupId>
    <artifactId>play_${scala.major.version}</artifactId>
    <version>${play2.version}</version>
    <exclusions>
        <exclusion>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_${scala.major.version}</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- AFTER -->
<dependency>
    <groupId>com.typesafe.play</groupId>
    <artifactId>play_${scala.major.version}</artifactId>
    <version>${play2.version}</version>
    <!-- Pekko is the default in Play 2.9, no exclusions needed -->
</dependency>
```

**Remove Akka Remote:**
```xml
<!-- REMOVE THIS -->
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-remote_${scala.major.version}</artifactId>
    <version>${typesafe.akka.version}</version>
</dependency>
```

**Add Pekko Remote:**
```xml
<!-- ADD THIS -->
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-remote_${scala.major.version}</artifactId>
    <version>${pekko.version}</version>
</dependency>
```

**Update Server Dependencies:**
```xml
<!-- BEFORE -->
<dependency>
    <groupId>com.typesafe.play</groupId>
    <artifactId>play-akka-http-server_${scala.major.version}</artifactId>
    <version>${play2.version}</version>
    <scope>runtime</scope>
</dependency>

<!-- AFTER -->
<dependency>
    <groupId>org.playframework</groupId>
    <artifactId>play-pekko-http-server_${scala.major.version}</artifactId>
    <version>${play2.version}</version>
    <scope>runtime</scope>
</dependency>
```

---

## 2. Java Source Code Changes

### 2.1 Actor Framework Core

#### File: sb-actor/src/main/java/org/sunbird/actor/core/ActorService.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;
import akka.routing.FromConfig;

// AFTER
import org.apache.pekko.actor.ActorRef;
import org.apache.pekko.actor.ActorSystem;
import org.apache.pekko.actor.Props;
import org.apache.pekko.routing.FromConfig;
```

**No other changes needed in this file.**

#### File: sb-actor/src/main/java/org/sunbird/actor/core/ActorCache.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorRef;

// AFTER
import org.apache.pekko.actor.ActorRef;
```

**No other changes needed in this file.**

### 2.2 Base Actor

#### File: all-actors/src/main/java/org/sunbird/BaseActor.java

**Import Changes:**
```java
// BEFORE
import akka.actor.UntypedAbstractActor;

// AFTER
import org.apache.pekko.actor.UntypedAbstractActor;
```

**No other changes needed in this file.**

### 2.3 Application Bootstrap

#### File: all-actors/src/main/java/org/sunbird/Application.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorRef;

// AFTER
import org.apache.pekko.actor.ActorRef;
```

**No other changes needed in this file.**

### 2.4 Business Logic Actors

All actor files in `/all-actors/src/main/java/org/sunbird/notification/actor/` and `/all-actors/src/main/java/org/sunbird/health/actor/`:

**Files:**
- NotificationActor.java
- CreateNotificationActor.java
- ReadNotificationActor.java
- UpdateNotificationActor.java
- DeleteNotificationActor.java
- NotificationTemplateActor.java
- HealthActor.java

**No imports to change** - These extend BaseActor, which already imports UntypedAbstractActor.

### 2.5 Service Layer

#### File: service/app/utils/module/SignalHandler.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorSystem;

// AFTER
import org.apache.pekko.actor.ActorSystem;
```

**No other changes needed in this file.**

#### File: service/app/controllers/BaseController.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorRef;
import akka.pattern.Patterns;
import akka.util.Timeout;

// AFTER
import org.apache.pekko.actor.ActorRef;
import org.apache.pekko.pattern.Patterns;
import org.apache.pekko.util.Timeout;
```

**No other changes needed in this file.**

#### File: service/app/controllers/ResponseHandler.java

**Import Changes:**
```java
// BEFORE
import akka.pattern.Patterns;
import akka.util.Timeout;

// AFTER
import org.apache.pekko.pattern.Patterns;
import org.apache.pekko.util.Timeout;
```

**No other changes needed in this file.**

### 2.6 Test Files

#### File: all-actors/src/test/java/org/sunbird/notification/actor/BaseActorTest.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorSystem;
import akka.testkit.javadsl.TestKit;

// AFTER
import org.apache.pekko.actor.ActorSystem;
import org.apache.pekko.testkit.javadsl.TestKit;
```

**No other changes needed in this file.**

#### File: all-actors/src/test/java/org/sunbird/notification/actor/NotificationActorTest.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorRef;
import akka.actor.Props;
import akka.testkit.javadsl.TestKit;

// AFTER
import org.apache.pekko.actor.ActorRef;
import org.apache.pekko.actor.Props;
import org.apache.pekko.testkit.javadsl.TestKit;
```

**Similar changes for:**
- CreateNotificationActorTest.java
- ReadNotificationActorTest.java
- UpdateNotificationActorTest.java
- DeleteNotificationActorTest.java
- NotificationTemplateActorTest.java

#### File: service/test/controllers/BaseControllerTest.java

**Import Changes:**
```java
// BEFORE
import akka.actor.ActorRef;

// AFTER
import org.apache.pekko.actor.ActorRef;
```

#### File: service/test/controllers/DummyActor.java

**Import Changes:**
```java
// BEFORE
import akka.actor.UntypedAbstractActor;

// AFTER
import org.apache.pekko.actor.UntypedAbstractActor;
```

---

## 3. Configuration File Changes

### 3.1 Play Application Configuration

#### File: service/conf/application.conf

**Section 1: Top-level Akka Configuration (Lines 5-33)**

**BEFORE:**
```hocon
## Akka
# https://www.playframework.com/documentation/latest/JavaAkka#Configuration
# ~~~~~
akka {
  stdout-loglevel = "OFF"
  loglevel = "OFF"
  log-config-on-start = off
  actor {
    default-dispatcher {
      fork-join-executor {
        parallelism-min = 8
        parallelism-factor = 32.0
        parallelism-max = 64
        task-peeking-mode = "FIFO"
      }
    }
  }
}
```

**AFTER:**
```hocon
## Pekko (formerly Akka)
# https://pekko.apache.org/docs/pekko/current/
# ~~~~~
pekko {
  stdout-loglevel = "OFF"
  loglevel = "OFF"
  log-config-on-start = off
  actor {
    default-dispatcher {
      fork-join-executor {
        parallelism-min = 8
        parallelism-factor = 32.0
        parallelism-max = 64
        task-peeking-mode = "FIFO"
      }
    }
  }
}
```

**Section 2: Actor System Configuration (Lines 140-227)**

**BEFORE:**
```hocon
notificationActorSystem {
  default-dispatcher {
    type = "Dispatcher"
    executor = "fork-join-executor"
    fork-join-executor {
      parallelism-min = 8
      parallelism-factor = 32.0
      parallelism-max = 64
    }
    throughput = 1
  }
  
  notification-dispatcher {
    type = "Dispatcher"
    executor = "fork-join-executor"
    fork-join-executor {
      parallelism-min = 8
      parallelism-factor = 32.0
      parallelism-max = 64
    }
    throughput = 1
  }

  akka {
    stdout-loglevel = "OFF"
    actor {
      akka.actor.allow-java-serialization = off
      
      default-dispatcher {
        type = "Dispatcher"
        executor = "fork-join-executor"
        fork-join-executor {
          parallelism-min = 8
          parallelism-factor = 32.0
          parallelism-max = 64
        }
        throughput = 1
      }
      
      deployment {
        /HealthActor {
          router = smallest-mailbox-pool
          nr-of-instances = 5
          dispatcher = default-dispatcher
        }
        /NotificationActor {
          router = smallest-mailbox-pool
          nr-of-instances = 10
          dispatcher = notification-dispatcher
        }
        # ... more actors ...
      }
    }
  }
  
  remote {
    maximum-payload-bytes = 30000000 bytes
    netty.tcp {
      port = 8088
      message-frame-size = 30000000b
      send-buffer-size = 30000000b
      receive-buffer-size = 30000000b
      maximum-frame-size = 30000000b
    }
  }
}
```

**AFTER:**
```hocon
notificationActorSystem {
  default-dispatcher {
    type = "Dispatcher"
    executor = "fork-join-executor"
    fork-join-executor {
      parallelism-min = 8
      parallelism-factor = 32.0
      parallelism-max = 64
    }
    throughput = 1
  }
  
  notification-dispatcher {
    type = "Dispatcher"
    executor = "fork-join-executor"
    fork-join-executor {
      parallelism-min = 8
      parallelism-factor = 32.0
      parallelism-max = 64
    }
    throughput = 1
  }

  pekko {
    stdout-loglevel = "OFF"
    actor {
      allow-java-serialization = off
      
      default-dispatcher {
        type = "Dispatcher"
        executor = "fork-join-executor"
        fork-join-executor {
          parallelism-min = 8
          parallelism-factor = 32.0
          parallelism-max = 64
        }
        throughput = 1
      }
      
      deployment {
        /HealthActor {
          router = smallest-mailbox-pool
          nr-of-instances = 5
          dispatcher = default-dispatcher
        }
        /NotificationActor {
          router = smallest-mailbox-pool
          nr-of-instances = 10
          dispatcher = notification-dispatcher
        }
        # ... more actors ...
      }
    }
    
    remote {
      artery {
        enabled = on
        transport = tcp
        canonical {
          hostname = "127.0.0.1"
          port = 8088
        }
        advanced {
          maximum-frame-size = 30000000b
          maximum-large-frame-size = 30 MiB
          buffer-pool-size = 128
          # Fine-tuning for large messages
          outbound-message-queue-size = 3072
          outbound-control-queue-size = 3072
        }
      }
    }
  }
}
```

**Key Changes:**
1. `akka` → `pekko` (namespace change)
2. `akka.actor.allow-java-serialization` → `allow-java-serialization` (moved up one level)
3. `remote.netty.tcp` → `remote.artery` (Artery is the new remoting implementation)
4. New Artery configuration structure with `canonical` hostname/port

### 3.2 Actor Module Configuration

#### File: all-actors/src/main/resources/application.conf

**Current:** (Empty file)

**No changes needed** - This file is empty currently.

---

## 4. Build and Deployment Files

### 4.1 Dockerfile

**File:** `/Dockerfile`

**No changes needed** - Dockerfile is JVM-agnostic. The compiled JAR will work the same way.

### 4.2 Jenkinsfile

**File:** `/Jenkinsfile`

**Potential Changes:**
- Build commands remain the same (`mvn clean install`)
- Test commands remain the same
- Package commands remain the same (`mvn play2:dist`)

**No changes needed** unless build tool is switched to SBT.

### 4.3 Build Script

**File:** `/build.sh`

**No changes needed** - Docker build script is framework-agnostic.

---

## 5. Automated Migration Script

Here's a bash script to automate the import changes:

```bash
#!/bin/bash
# Automated Akka to Pekko import replacement script

echo "Starting Akka to Pekko migration..."

# Find all Java files with Akka imports
find . -name "*.java" -type f -exec grep -l "import akka\." {} \; > /tmp/files_to_migrate.txt

# Replace imports in each file
while IFS= read -r file; do
    echo "Processing: $file"
    
    # Backup original file
    cp "$file" "$file.bak"
    
    # Replace imports
    sed -i 's/import akka\./import org.apache.pekko./g' "$file"
    
    echo "  ✓ Updated imports"
done < /tmp/files_to_migrate.txt

echo ""
echo "Migration complete!"
echo "Files processed: $(wc -l < /tmp/files_to_migrate.txt)"
echo ""
echo "Backup files created with .bak extension"
echo "Please review changes and test before committing"
```

**Usage:**
```bash
chmod +x migrate_imports.sh
./migrate_imports.sh
```

---

## 6. Testing Checklist

### 6.1 After Each File Change

- [ ] Code compiles without errors
- [ ] No deprecation warnings
- [ ] IDE shows no errors

### 6.2 After All Changes

- [ ] `mvn clean compile` succeeds
- [ ] `mvn test` passes all tests
- [ ] Application starts successfully
- [ ] Health endpoint responds
- [ ] Actor communication works
- [ ] Remote actors communicate properly
- [ ] Performance meets baseline

### 6.3 Comprehensive Testing

- [ ] Unit tests pass (100%)
- [ ] Integration tests pass
- [ ] Actor message passing works
- [ ] Router pooling works correctly
- [ ] Dispatcher configuration applies
- [ ] Remote actor discovery works
- [ ] Message serialization works
- [ ] Error handling works
- [ ] Timeouts configured correctly
- [ ] Performance benchmarks met

---

## 7. Rollback Procedure

If issues are encountered:

1. **Code Rollback:**
   ```bash
   git reset --hard HEAD~1  # or specific commit
   ```

2. **Dependency Rollback:**
   - Revert all pom.xml files
   - Run `mvn clean install`

3. **Configuration Rollback:**
   - Revert application.conf files
   - Restart services

4. **Verification:**
   - Run test suite
   - Verify health endpoint
   - Check logs for errors

---

## 8. Quick Reference Tables

### 8.1 Package Mapping

| Akka Package | Pekko Package |
|-------------|---------------|
| `akka.actor.*` | `org.apache.pekko.actor.*` |
| `akka.pattern.*` | `org.apache.pekko.pattern.*` |
| `akka.routing.*` | `org.apache.pekko.routing.*` |
| `akka.testkit.*` | `org.apache.pekko.testkit.*` |
| `akka.util.*` | `org.apache.pekko.util.*` |

### 8.2 Maven Coordinates Mapping

| Akka Artifact | Pekko Artifact |
|--------------|----------------|
| `com.typesafe.akka:akka-actor_2.12` | `org.apache.pekko:pekko-actor_2.13` |
| `com.typesafe.akka:akka-remote_2.12` | `org.apache.pekko:pekko-remote_2.13` |
| `com.typesafe.akka:akka-testkit_2.12` | `org.apache.pekko:pekko-testkit_2.13` |
| `com.typesafe.akka:akka-stream_2.12` | `org.apache.pekko:pekko-stream_2.13` |
| `com.typesafe.play:play-akka-http-server_2.12` | `org.playframework:play-pekko-http-server_2.13` |

### 8.3 Configuration Mapping

| Akka Config | Pekko Config |
|------------|--------------|
| `akka.actor.*` | `pekko.actor.*` |
| `akka.remote.netty.tcp.*` | `pekko.remote.artery.canonical.*` |
| `akka.actor.allow-java-serialization` | `pekko.actor.allow-java-serialization` |

---

## 9. Common Pitfalls and Solutions

### Issue 1: Scala Version Mismatch
**Problem:** Mixing Scala 2.12 and 2.13 dependencies
**Solution:** Ensure ALL dependencies use `_2.13` suffix

### Issue 2: Artery Configuration
**Problem:** Old `netty.tcp` config doesn't work
**Solution:** Follow new Artery config structure in section 3.1

### Issue 3: Serialization Issues
**Problem:** Messages fail to serialize with Artery
**Solution:** Ensure `allow-java-serialization = off` and use proper serializers

### Issue 4: Remote Actor Connection Fails
**Problem:** Actors can't connect remotely
**Solution:** Check `canonical.hostname` and `canonical.port` in Artery config

### Issue 5: Test Failures
**Problem:** Tests fail after migration
**Solution:** Update test configuration to use Pekko TestKit properly

---

## 10. Support and Resources

### Documentation
- **Pekko Docs:** https://pekko.apache.org/docs/pekko/current/
- **Play 2.9 Docs:** https://www.playframework.com/documentation/2.9.x/
- **Migration Guide:** https://pekko.apache.org/docs/pekko/current/project/migration-guides.html

### Community
- **Pekko GitHub:** https://github.com/apache/incubator-pekko
- **Play GitHub:** https://github.com/playframework/playframework
- **Stack Overflow:** Tag `apache-pekko`

### Contact
- **For questions:** Refer to the full compatibility report
- **For issues:** Create detailed bug reports with logs

---

**Document Version:** 1.0  
**Last Updated:** October 7, 2025  
**Status:** Ready for Implementation  

**Related Documents:**
- [MIGRATION_COMPATIBILITY_REPORT.md](./MIGRATION_COMPATIBILITY_REPORT.md) - Full analysis
- [MIGRATION_SUMMARY.md](./MIGRATION_SUMMARY.md) - Executive summary
