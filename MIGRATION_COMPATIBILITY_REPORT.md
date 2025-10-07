# Play Framework Upgrade & Akka to Pekko Migration - Compatibility Report

## Executive Summary

This report analyzes the Sunbird Notification Service repository for upgrading Play Framework and migrating from Akka to Apache Pekko. The analysis covers current dependencies, migration paths, potential issues, drawbacks, and benefits.

**Current State:**
- **Play Framework:** 2.7.2 (Released April 2019, EOL)
- **Akka:** 2.5.22 (Pre-BSL license change)
- **Scala:** 2.12.11
- **Java:** 11
- **Build Tool:** Maven with play2-maven-plugin

**License Change Context:**
Lightbend changed Akka's license from Apache 2.0 to Business Source License (BSL) 1.1 starting with Akka 2.7+ (September 2022). Apache Pekko is a fork of Akka 2.6.x maintained by the Apache Software Foundation under Apache 2.0 license.

---

## Table of Contents
1. [Current Architecture Analysis](#current-architecture-analysis)
2. [Play Framework Upgrade Analysis](#play-framework-upgrade-analysis)
3. [Akka to Pekko Migration Analysis](#akka-to-pekko-migration-analysis)
4. [Migration Strategy](#migration-strategy)
5. [Issues and Challenges](#issues-and-challenges)
6. [Drawbacks](#drawbacks)
7. [Benefits](#benefits)
8. [Recommendations](#recommendations)

---

## 1. Current Architecture Analysis

### 1.1 Project Structure
```
notification-service/
├── service/              # Play Framework application (Maven play2 packaging)
├── all-actors/          # Business logic actors
├── sb-actor/            # Actor framework core
├── sb-utils/            # Utilities
├── notification-sdk/    # SDK module
├── cassandra-utils/     # Database utilities
├── sb-telemetry-utils/  # Telemetry utilities
├── sb-common/           # Common utilities
└── reports/             # Reporting module
```

### 1.2 Current Dependencies

#### Play Framework Dependencies:
```xml
<play2.version>2.7.2</play2.version>
<play2.plugin.version>1.0.0-rc5</play2.plugin.version>

Dependencies:
- com.typesafe.play:play_2.12:2.7.2
- com.typesafe.play:play-guice_2.12:2.7.2
- com.typesafe.play:play-akka-http-server_2.12:2.7.2
- com.typesafe.play:play-netty-server_2.12:2.7.2
- com.typesafe.play:filters-helpers_2.12:2.7.2
- com.typesafe.play:play-test_2.12:2.7.2
- com.typesafe.play:play-logback_2.12:2.7.2
```

#### Akka Dependencies:
```xml
<typesafe.akka.version>2.5.22</typesafe.akka.version>

Dependencies:
- com.typesafe.akka:akka-actor_2.12:2.5.22
- com.typesafe.akka:akka-remote_2.12:2.5.22
- com.typesafe.akka:akka-testkit_2.12:2.5.22
- com.typesafe.akka:akka-http-core_2.12:10.1.8 (transitive via Play)
- com.typesafe.akka:akka-stream_2.12:2.5.22 (transitive)
- com.typesafe.akka:akka-slf4j_2.12:2.5.22 (transitive)
```

#### Scala Dependencies:
```xml
<scala.version>2.12.11</scala.version>
<scala.major.version>2.12</scala.major.version>
```

### 1.3 Akka Usage Analysis

**Files Using Akka (18 Java files):**

1. **Core Actor Classes:**
   - `/sb-actor/src/main/java/org/sunbird/actor/core/ActorService.java`
   - `/sb-actor/src/main/java/org/sunbird/actor/core/ActorCache.java`
   - `/sb-actor/src/main/java/org/sunbird/actor/core/ActorConfig.java`

2. **Base Actor:**
   - `/all-actors/src/main/java/org/sunbird/BaseActor.java`

3. **Business Logic Actors:**
   - `/all-actors/src/main/java/org/sunbird/notification/actor/NotificationActor.java`
   - `/all-actors/src/main/java/org/sunbird/notification/actor/CreateNotificationActor.java`
   - `/all-actors/src/main/java/org/sunbird/notification/actor/ReadNotificationActor.java`
   - `/all-actors/src/main/java/org/sunbird/notification/actor/UpdateNotificationActor.java`
   - `/all-actors/src/main/java/org/sunbird/notification/actor/DeleteNotificationActor.java`
   - `/all-actors/src/main/java/org/sunbird/notification/actor/NotificationTemplateActor.java`
   - `/all-actors/src/main/java/org/sunbird/health/actor/HealthActor.java`

4. **Service Layer:**
   - `/service/app/utils/module/SignalHandler.java`
   - `/service/app/controllers/BaseController.java`
   - `/service/app/controllers/ResponseHandler.java`

5. **Application Bootstrap:**
   - `/all-actors/src/main/java/org/sunbird/Application.java`

6. **Test Classes:**
   - Multiple test files in `all-actors/src/test/java/` and `service/test/`

**Akka Imports Used:**
```java
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;
import akka.actor.UntypedAbstractActor;
import akka.pattern.Patterns;
import akka.routing.FromConfig;
import akka.testkit.javadsl.TestKit;
import akka.util.Timeout;
```

**Actor Features Used:**
- Actor System initialization
- Actor creation with Props
- Router configuration (FromConfig, smallest-mailbox-pool)
- Remote actors (Akka Remote)
- Custom dispatchers
- Actor routing and pooling
- Ask pattern (Patterns.ask)
- TestKit for testing

### 1.4 Configuration Files

**application.conf:**
- Located in `/service/conf/application.conf` and `/all-actors/src/main/resources/application.conf`
- Contains Akka-specific configuration:
  - Actor system settings
  - Dispatcher configuration
  - Router deployment settings
  - Remote actor configuration
  - Fork-join executor settings

---

## 2. Play Framework Upgrade Analysis

### 2.1 Current Version Analysis

**Play 2.7.2 (April 2019):**
- Based on Akka 2.5.x
- Scala 2.12 compatible
- Java 8+ support
- End of Life: Already EOL

### 2.2 Upgrade Path Options

#### Option 1: Play 2.8.x (Last Akka-based version)
**Play 2.8.22 (June 2023) - Final Release:**
- **Akka Version:** 2.6.21 (Last Apache 2.0 version)
- **Scala:** 2.12 or 2.13
- **Java:** 8, 11
- **Status:** End of Life (November 2023)
- **License:** Apache 2.0 (since using Akka 2.6.x)

**Changes from 2.7:**
- Updated to Akka 2.6.x
- Better Java 11 support
- Improved Guice integration
- Security updates
- Performance improvements

**Breaking Changes:**
- Some configuration changes
- Updated APIs for forms
- Changes to WebSocket handling
- Deprecated APIs removed

#### Option 2: Play 2.9.x (First Pekko-based version)
**Play 2.9.5 (Latest as of 2024):**
- **Actor Framework:** Apache Pekko 1.0.x (fork of Akka 2.6.x)
- **Scala:** 2.13 or 3.x
- **Java:** 11, 17, 21
- **Status:** Active maintenance
- **License:** Apache 2.0

**Major Changes from 2.8:**
- Complete migration from Akka to Pekko
- All `com.typesafe.akka` imports replaced with `org.apache.pekko`
- Requires Scala 2.13+ (no Scala 2.12 support)
- Requires Java 11+ minimum
- Updated HTTP server components
- New package structure for Pekko

**Breaking Changes:**
- **ALL** Akka references must be changed to Pekko
- Scala 2.13 is minimum (requires cross-compilation)
- Package namespace changes: `akka.*` → `pekko.*`
- Configuration namespace changes: `akka.*` → `pekko.*`
- Different Maven coordinates for dependencies

#### Option 3: Play 3.0.x (Latest)
**Play 3.0.x (2024):**
- **Actor Framework:** Apache Pekko 1.1.x
- **Scala:** 2.13, 3.3+
- **Java:** 11, 17, 21
- **Status:** Latest stable
- **License:** Apache 2.0

**Major Changes:**
- Same Pekko migration as 2.9.x
- Additional API modernization
- Improved performance
- Enhanced security
- Better Java 17+ integration

### 2.3 Build Tool Considerations

**Current: play2-maven-plugin 1.0.0-rc5**
- Supports Play 2.7.x
- Last updated 2019
- Limited to Scala 2.12

**For Play 2.8+:**
- Same plugin should work with minor configuration changes
- May need version update

**For Play 2.9+/3.0+:**
- Major plugin changes required
- New Maven coordinates
- Configuration restructuring
- SBT is better supported for Play 2.9+

---

## 3. Akka to Pekko Migration Analysis

### 3.1 Apache Pekko Overview

**What is Pekko?**
- Fork of Akka 2.6.x by Apache Software Foundation
- Created after Lightbend's license change
- Maintains Apache 2.0 license
- API-compatible with Akka 2.6.x with namespace changes
- Active development and community support

**Current Versions:**
- **Pekko 1.0.x:** Fork of Akka 2.6.x (Stable)
- **Pekko 1.1.x:** Latest with additional improvements (Stable)

**Key Differences:**
- Package namespace: `com.typesafe.akka` → `org.apache.pekko`
- Maven group ID: `com.typesafe.akka` → `org.apache.pekko`
- Configuration namespace: `akka.*` → `pekko.*`
- Otherwise API-compatible

### 3.2 Migration Scope

#### 3.2.1 Code Changes Required

**1. Import Statements (36 occurrences across 18 files):**
```java
// Before (Akka)
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;
import akka.actor.UntypedAbstractActor;
import akka.pattern.Patterns;
import akka.routing.FromConfig;
import akka.testkit.javadsl.TestKit;
import akka.util.Timeout;

// After (Pekko)
import org.apache.pekko.actor.ActorRef;
import org.apache.pekko.actor.ActorSystem;
import org.apache.pekko.actor.Props;
import org.apache.pekko.actor.UntypedAbstractActor;
import org.apache.pekko.pattern.Patterns;
import org.apache.pekko.routing.FromConfig;
import org.apache.pekko.testkit.javadsl.TestKit;
import org.apache.pekko.util.Timeout;
```

**2. Class Inheritance:**
```java
// Before
public abstract class BaseActor extends UntypedAbstractActor { }

// After
public abstract class BaseActor extends org.apache.pekko.actor.UntypedAbstractActor { }
```

**3. Configuration Files:**

`application.conf` - Two files need updates:
- `/service/conf/application.conf`
- `/all-actors/src/main/resources/application.conf`

```hocon
# Before
akka {
  stdout-loglevel = "OFF"
  loglevel = "OFF"
  actor {
    default-dispatcher { ... }
  }
}

notificationActorSystem {
  akka {
    actor { ... }
  }
  remote {
    netty.tcp { ... }
  }
}

# After
pekko {
  stdout-loglevel = "OFF"
  loglevel = "OFF"
  actor {
    default-dispatcher { ... }
  }
}

notificationActorSystem {
  pekko {
    actor { ... }
  }
  remote {
    artery.tcp { ... }  # Note: Pekko uses artery, not netty.tcp
  }
}
```

**4. Maven Dependencies:**

`pom.xml` (parent and modules):
```xml
<!-- Before (Akka) -->
<typesafe.akka.version>2.5.22</typesafe.akka.version>

<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor_${scala.major.version}</artifactId>
    <version>${typesafe.akka.version}</version>
</dependency>

<!-- After (Pekko) -->
<pekko.version>1.0.3</pekko.version>

<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-actor_${scala.major.version}</artifactId>
    <version>${pekko.version}</version>
</dependency>
```

**Required Pekko Dependencies:**
- `org.apache.pekko:pekko-actor_2.13`
- `org.apache.pekko:pekko-remote_2.13`
- `org.apache.pekko:pekko-testkit_2.13` (test scope)
- `org.apache.pekko:pekko-slf4j_2.13`
- `org.apache.pekko:pekko-stream_2.13`

#### 3.2.2 Files Requiring Changes

**Java Source Files (18 files):**
1. Actor framework core (3 files in sb-actor)
2. Base actor (1 file in all-actors)
3. Business actors (6 files in all-actors)
4. Service layer (3 files in service)
5. Test files (5 files)

**Configuration Files (2 files):**
1. `/service/conf/application.conf`
2. `/all-actors/src/main/resources/application.conf`

**Build Files (9 pom.xml files):**
1. Root `pom.xml`
2. `service/pom.xml`
3. `all-actors/pom.xml`
4. `sb-actor/pom.xml`
5. Plus other module pom.xml files

### 3.3 Compatibility Matrix

| Component | Current (Akka) | Target (Pekko) | Compatibility |
|-----------|----------------|----------------|---------------|
| Actor API | Akka 2.5.22 | Pekko 1.0.3 | ✅ API Compatible (with namespace change) |
| Remote Actors | akka-remote | pekko-remote | ⚠️ Configuration changes needed (artery) |
| Routing | akka.routing | pekko.routing | ✅ Compatible |
| Testing | akka-testkit | pekko-testkit | ✅ Compatible |
| Streams | akka-stream | pekko-stream | ✅ Compatible |
| HTTP | akka-http | pekko-http | ✅ Compatible |
| Scala Version | 2.12.x | 2.13.x required | ❌ Requires upgrade |
| Java Version | 11 | 11, 17, 21 | ✅ Compatible |

### 3.4 Remote Actor Considerations

**Current Configuration (Akka Remote with Netty):**
```hocon
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
```

**Pekko Remote (Uses Artery):**
```hocon
remote {
  artery {
    enabled = on
    transport = tcp  # or aeron-udp
    canonical.hostname = "127.0.0.1"
    canonical.port = 8088
    
    advanced {
      maximum-frame-size = 30000000b
      buffer-pool-size = 128
      maximum-large-frame-size = 30 MiB
    }
  }
}
```

**Impact:**
- Artery is more performant than classic remoting
- Configuration structure is different
- Testing required for remote actor communication

---

## 4. Migration Strategy

### 4.1 Phased Approach

#### Phase 1: Preparation (No Code Changes)
**Duration:** 1-2 weeks
**Activities:**
1. ✅ Complete this analysis (Current)
2. Set up test environment
3. Create comprehensive test suite for current functionality
4. Document current actor behavior and communication patterns
5. Create rollback plan
6. Backup current working state

**Deliverables:**
- This compatibility report
- Test coverage report
- Migration runbook
- Rollback procedure

#### Phase 2: Scala Upgrade (If needed for Pekko)
**Duration:** 2-3 weeks
**Activities:**
1. Upgrade Scala from 2.12.11 to 2.13.x
2. Update all Scala dependencies
3. Recompile and test
4. Fix any Scala 2.13 compatibility issues

**Risk:** Medium - Some libraries may not have Scala 2.13 versions

#### Phase 3: Play Framework Upgrade
**Duration:** 2-4 weeks

**Option A: Incremental (Recommended)**
```
Current (2.7.2) → Play 2.8.22 → Play 2.9.x
```
- Safer with validation at each step
- Stays on Akka 2.6 (Apache 2.0) in intermediate step
- More time-consuming

**Option B: Direct (Risky)**
```
Current (2.7.2) → Play 2.9.x directly
```
- Faster but riskier
- Combines multiple breaking changes
- Harder to debug issues

**Activities for Play Upgrade:**
1. Update Play dependencies
2. Update play2-maven-plugin (or migrate to SBT if needed)
3. Fix deprecated API usage
4. Update route files if needed
5. Update form handling
6. Test all endpoints
7. Performance testing

#### Phase 4: Akka to Pekko Migration
**Duration:** 3-4 weeks

**Step 4.1: Dependency Migration**
1. Update parent pom.xml
2. Replace Akka dependencies with Pekko
3. Ensure version compatibility

**Step 4.2: Code Migration**
1. Global search-replace for imports:
   - `import akka.` → `import org.apache.pekko.`
2. Update class references
3. Fix any API differences

**Step 4.3: Configuration Migration**
1. Update application.conf files
2. Migrate from netty.tcp to artery
3. Update dispatcher configurations
4. Update router configurations

**Step 4.4: Testing**
1. Unit tests
2. Integration tests
3. Actor communication tests
4. Remote actor tests
5. Performance tests
6. Load tests

#### Phase 5: Validation and Optimization
**Duration:** 2-3 weeks
**Activities:**
1. Full regression testing
2. Performance benchmarking
3. Load testing
4. Security testing
5. Documentation updates
6. Deployment testing
7. Monitoring setup

### 4.2 Alternative Approach: Parallel Development

Create a feature branch with all migrations and maintain both versions:
- **Pros:** Faster migration, easier A/B comparison
- **Cons:** More merge conflicts, harder to maintain, resource intensive

### 4.3 Minimal Risk Approach (Stay on Current Stack)

**Option:** Freeze at Play 2.8.22 + Akka 2.6.21
- **Pros:** 
  - Minimal changes
  - Stays Apache 2.0 licensed
  - Proven stable
- **Cons:**
  - No long-term support
  - Security vulnerabilities not patched
  - Technical debt accumulation
- **Recommended for:** Short-term (6-12 months maximum)

---

## 5. Issues and Challenges

### 5.1 Technical Challenges

#### 5.1.1 Scala Version Incompatibility
**Issue:** Pekko 1.0.x requires Scala 2.13+, current project uses Scala 2.12.11

**Impact:** High
**Complexity:** Medium-High

**Details:**
- All Scala dependencies must be recompiled for 2.13
- Some libraries may not have Scala 2.13 versions
- Binary incompatibility between Scala 2.12 and 2.13
- Cross-compilation not possible (can't mix versions)

**Affected Components:**
- All Scala-based dependencies
- Play framework artifacts
- Custom Scala code (if any)
- Third-party libraries

**Mitigation:**
- Audit all dependencies for Scala 2.13 availability
- Find alternatives for libraries without 2.13 support
- Test thoroughly after upgrade

#### 5.1.2 Play Maven Plugin Limitations
**Issue:** play2-maven-plugin may not fully support Play 2.9+

**Impact:** High
**Complexity:** High

**Details:**
- Current plugin version: 1.0.0-rc5 (from 2019)
- Limited documentation for newer Play versions
- May require migration to SBT (Scala Build Tool)

**Alternative Solutions:**
1. Update to latest play2-maven-plugin (if available)
2. Migrate to SBT (Play's recommended build tool)
3. Use custom Maven configuration with exec-maven-plugin

**SBT Migration Considerations:**
- Complete build restructure required
- Jenkins pipeline changes needed
- Learning curve for team
- Better Play ecosystem support
- More documentation available

#### 5.1.3 Remote Actor Configuration Changes
**Issue:** Pekko Remote uses Artery instead of Netty TCP

**Impact:** Medium
**Complexity:** Medium

**Details:**
- Configuration structure completely different
- Different performance characteristics
- Requires extensive testing
- May affect distributed system behavior

**Current Configuration:**
```hocon
remote.netty.tcp {
  port = 8088
  message-frame-size = 30000000b
  send-buffer-size = 30000000b
  receive-buffer-size = 30000000b
}
```

**New Configuration Required:**
```hocon
remote.artery {
  canonical.port = 8088
  advanced.maximum-frame-size = 30000000b
  # Different configuration model
}
```

**Risks:**
- Message serialization differences
- Performance impact (could be positive or negative)
- Network behavior changes
- Debugging complexity

#### 5.1.4 API Deprecations and Breaking Changes
**Issue:** APIs deprecated in Akka 2.5 → 2.6 and removed in Pekko

**Impact:** Medium
**Complexity:** Low-Medium

**Known Deprecations:**
- Some router APIs
- Configuration key names
- Pattern matching in actors
- Futures API changes

**Mitigation:**
- Compile with deprecation warnings
- Update to recommended APIs
- Comprehensive testing

#### 5.1.5 Testing Infrastructure
**Issue:** Test framework changes required

**Impact:** Medium
**Complexity:** Medium

**Details:**
- akka-testkit → pekko-testkit
- Test configuration changes
- May require test rewrite for edge cases
- PowerMock compatibility issues with newer Java versions

**Current Test Dependencies:**
```xml
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-testkit_2.12</artifactId>
    <version>2.5.22</version>
    <scope>test</scope>
</dependency>
```

**New Dependencies:**
```xml
<dependency>
    <groupId>org.apache.pekko</groupId>
    <artifactId>pekko-testkit_2.13</artifactId>
    <version>1.0.3</version>
    <scope>test</scope>
</dependency>
```

#### 5.1.6 Third-Party Dependencies
**Issue:** Some dependencies may not support Pekko yet

**Impact:** Medium-High
**Complexity:** Variable

**Potentially Affected:**
- Akka HTTP-based libraries
- Monitoring/metrics libraries (if Akka-specific)
- Custom plugins
- Integration libraries

**Investigation Required:**
- Audit all dependencies
- Check Pekko compatibility
- Find alternatives if needed

### 5.2 Operational Challenges

#### 5.2.1 Development Environment
**Impact:** Low-Medium
**Challenge:** All developers need updated environments
**Time Required:** 1-2 days per developer

**Requirements:**
- Updated Java (11+ already met)
- Updated Maven settings
- New dependency downloads
- IDE configuration updates

#### 5.2.2 CI/CD Pipeline
**Impact:** Medium
**Challenge:** Jenkins pipeline updates required

**Required Changes:**
- Updated build scripts
- Docker image updates
- Deployment configuration
- Test execution changes

**Files Affected:**
- `Jenkinsfile`
- `Dockerfile`
- `build.sh`
- Maven settings

#### 5.2.3 Monitoring and Observability
**Impact:** Medium
**Challenge:** Monitoring tools may need updates

**Concerns:**
- JMX metrics may change
- Telemetry configuration
- Log format changes
- Performance metrics

#### 5.2.4 Documentation and Knowledge Transfer
**Impact:** Medium
**Challenge:** Team training required

**Requirements:**
- Update internal documentation
- Team training on Pekko
- Update deployment guides
- New troubleshooting guides

### 5.3 Business Challenges

#### 5.3.1 Development Freeze
**Impact:** High
**Duration:** 6-12 weeks total migration time

**Considerations:**
- Feature development slowdown
- Resource allocation
- Parallel maintenance of old/new versions

#### 5.3.2 Testing Effort
**Impact:** High
**Effort:** Significant QA resources required

**Required Testing:**
- Unit tests
- Integration tests
- Performance tests
- Load tests
- Security tests
- User acceptance tests

#### 5.3.3 Rollback Planning
**Impact:** High
**Complexity:** High

**Challenges:**
- Database compatibility
- API compatibility
- Configuration rollback
- Deployment complexity

---

## 6. Drawbacks

### 6.1 Migration Effort and Cost

**Time Investment:**
- **Minimum:** 8-12 weeks
- **Realistic:** 12-16 weeks
- **Conservative:** 16-20 weeks

**Resource Requirements:**
- 2-3 full-time developers
- 1 QA engineer
- DevOps engineer support
- Project manager oversight

**Cost Breakdown:**
| Phase | Effort (person-weeks) | Risk Level |
|-------|----------------------|------------|
| Preparation | 1-2 | Low |
| Scala Upgrade | 2-3 | Medium |
| Play Upgrade | 2-4 | Medium-High |
| Pekko Migration | 3-4 | Medium |
| Testing & Validation | 2-3 | Medium |
| **Total** | **10-16** | **Medium-High** |

### 6.2 Technical Debt Increase (Short Term)

**During Migration:**
- Mixed codebase state
- Increased complexity
- More merge conflicts
- Harder to onboard new developers

### 6.3 Risk of Regression

**Potential Issues:**
- Subtle behavior changes
- Performance degradation
- New bugs introduced
- Edge cases missed

**High-Risk Areas:**
- Remote actor communication
- Message serialization
- Dispatcher behavior
- Router configuration

### 6.4 Ecosystem Maturity

**Pekko Considerations:**
- Smaller community than Akka
- Less documentation
- Fewer Stack Overflow answers
- Newer ecosystem (less proven in production)

**Play 2.9+ Considerations:**
- Newer version, less battle-tested
- Fewer production deployments
- Potential undiscovered issues

### 6.5 Build Tool Limitations

**Maven Challenges:**
- Limited Play 2.9+ support
- May require SBT migration
- Additional learning curve

**If SBT Migration Needed:**
- Complete build system change
- CI/CD pipeline overhaul
- New build files (build.sbt, project/*.scala)
- Different dependency management
- Team training required

### 6.6 Dependency Management

**Challenges:**
- Version conflicts possible
- Some libraries may lag in Pekko support
- Transitive dependency issues
- Need to find Scala 2.13 versions of all deps

### 6.7 Performance Unknowns

**Concerns:**
- Artery vs Netty performance differences
- Potential memory usage changes
- Throughput variations
- Latency changes

**Requires:**
- Extensive benchmarking
- Load testing
- Performance tuning
- Monitoring setup

### 6.8 Breaking Changes in Application Behavior

**Play Framework Changes:**
- Request handling differences
- Form validation changes
- JSON serialization variations
- Error handling modifications

**Pekko Changes:**
- Message ordering guarantees
- Supervision strategy subtle differences
- Serialization behavior
- Remote communication protocol

---

## 7. Benefits

### 7.1 License Compliance

**Primary Benefit: Avoid BSL License Issues**

**Current Risk:**
- Akka 2.7+ uses Business Source License (BSL) 1.1
- Not truly open source
- Restrictions on commercial use
- Potential legal implications

**With Pekko:**
- ✅ Apache 2.0 license (fully open source)
- ✅ No usage restrictions
- ✅ Commercial-friendly
- ✅ Community-driven governance
- ✅ Long-term sustainability

**Business Impact:**
- No licensing costs
- No legal compliance overhead
- Freedom to modify and distribute
- Better for enterprise adoption

### 7.2 Community Support and Sustainability

**Apache Software Foundation Benefits:**
- Established governance model
- Community-driven development
- Transparent decision-making
- Multiple vendor support
- Long-term project sustainability

**Pekko Community:**
- Growing adoption
- Active development
- Multiple contributors
- Regular releases
- Good documentation (improving)

### 7.3 Technical Improvements

#### 7.3.1 Performance Enhancements

**Artery (Pekko Remote):**
- ✅ Better throughput than classic remoting
- ✅ Lower latency
- ✅ More efficient serialization
- ✅ Better handling of large messages
- ✅ Improved back-pressure

**Play 2.9+:**
- ✅ Better HTTP/2 support
- ✅ Improved WebSocket handling
- ✅ Enhanced JSON performance
- ✅ Optimized routing

#### 7.3.2 Modern Java Support

**Java 11, 17, 21 Support:**
- ✅ Latest JVM optimizations
- ✅ Better GC options
- ✅ Security improvements
- ✅ Performance enhancements
- ✅ Modern language features

#### 7.3.3 Security Updates

**Active Maintenance:**
- ✅ Regular security patches
- ✅ CVE fixes
- ✅ Dependency updates
- ✅ Long-term support

**Current Stack (EOL):**
- ❌ No security updates
- ❌ Known vulnerabilities accumulate
- ❌ Compliance issues

### 7.4 Code Modernization

**Updated APIs:**
- ✅ Modern async patterns
- ✅ Better error handling
- ✅ Improved type safety
- ✅ Cleaner code structure

**Technical Debt Reduction:**
- Remove deprecated API usage
- Update to modern patterns
- Better code maintainability
- Easier onboarding

### 7.5 Better Tooling and Documentation

**Play 2.9+:**
- ✅ Improved dev mode
- ✅ Better error messages
- ✅ Enhanced debugging
- ✅ More comprehensive documentation

**Pekko:**
- ✅ Migration guides available
- ✅ API documentation
- ✅ Example projects
- ✅ Community support channels

### 7.6 Future-Proofing

**Long-Term Benefits:**
- ✅ Active development roadmap
- ✅ Modern architecture
- ✅ Community backing
- ✅ Easier future upgrades
- ✅ Better ecosystem integration

**Avoiding Technical Debt:**
- Staying on EOL software increases debt
- Migration becomes harder over time
- Team knowledge becomes outdated

### 7.7 Ecosystem Alignment

**Industry Trend:**
- Many projects migrating to Pekko
- Growing Pekko ecosystem
- Better integration with modern tools
- Alignment with open-source principles

### 7.8 Development Experience

**Developer Benefits:**
- ✅ Modern IDE support
- ✅ Better debugging tools
- ✅ Improved error messages
- ✅ More learning resources (over time)

### 7.9 Operational Benefits

**Deployment:**
- ✅ Better containerization support
- ✅ Cloud-native improvements
- ✅ Kubernetes integration
- ✅ Microservices patterns

**Monitoring:**
- ✅ Better metrics
- ✅ Enhanced observability
- ✅ Improved logging
- ✅ Distributed tracing support

### 7.10 Cost Savings

**Long-Term Savings:**
- No Akka BSL licensing costs (if applicable)
- Reduced security incident costs
- Lower maintenance overhead
- Better team productivity

**Competitive Advantage:**
- Modern tech stack attracts talent
- Faster feature development
- Better system reliability
- Improved performance

---

## 8. Recommendations

### 8.1 Primary Recommendation: Proceed with Migration

**Verdict: ✅ RECOMMENDED**

**Rationale:**
1. **License Compliance:** Avoid potential BSL license issues
2. **Security:** Current stack is EOL with no security updates
3. **Sustainability:** Apache-backed project with community support
4. **Future-Proofing:** Aligns with industry trends

**Recommended Timeline:** 12-16 weeks

### 8.2 Recommended Migration Path

**Phased Approach (Recommended):**

```
┌─────────────────────────────────────────────────────────────────┐
│ Phase 1: Preparation (1-2 weeks)                                │
│ - Complete analysis ✅                                           │
│ - Set up test environment                                       │
│ - Create comprehensive tests                                    │
│ - Document current behavior                                     │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 2: Scala Upgrade (2-3 weeks)                             │
│ - Upgrade to Scala 2.13.x                                       │
│ - Update all Scala dependencies                                 │
│ - Test and validate                                             │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 3: Play Framework Upgrade (2-4 weeks)                    │
│ Option A: 2.7.2 → 2.8.22 → 2.9.x (Safer)                      │
│ Option B: 2.7.2 → 2.9.x (Faster, riskier)                     │
│ - Update dependencies                                           │
│ - Fix breaking changes                                          │
│ - Test thoroughly                                               │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 4: Akka to Pekko Migration (3-4 weeks)                   │
│ - Update dependencies to Pekko                                  │
│ - Search-replace imports (akka → pekko)                        │
│ - Update configuration files                                    │
│ - Migrate remote actor config to Artery                         │
│ - Comprehensive testing                                         │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ Phase 5: Validation and Optimization (2-3 weeks)               │
│ - Performance benchmarking                                      │
│ - Load testing                                                  │
│ - Security testing                                              │
│ - Documentation updates                                         │
│ - Team training                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Total Duration:** 10-16 weeks

### 8.3 Target Versions

**Recommended Stack:**
- **Play Framework:** 2.9.5 (or latest 2.9.x)
- **Apache Pekko:** 1.0.3 (or latest 1.0.x)
- **Scala:** 2.13.14 (or latest 2.13.x)
- **Java:** 11 (current) or upgrade to 17
- **Build Tool:** Maven (keep current) or migrate to SBT (optional)

**Why not Play 3.0?**
- 2.9.x is more mature and stable
- Smaller migration step
- Can upgrade to 3.0 later if needed
- Better for risk mitigation

### 8.4 Build Tool Decision

**Recommendation: Stay with Maven (Short Term)**
- Lower risk
- Less change for team
- Existing expertise

**Consider SBT Migration (Long Term):**
- Better Play ecosystem support
- Official Play build tool
- Better incremental compilation
- Can be done as separate initiative

### 8.5 Risk Mitigation Strategies

#### 8.5.1 Comprehensive Testing Strategy
```
Unit Tests → Integration Tests → System Tests → Performance Tests → UAT
```

**Coverage Requirements:**
- ≥80% code coverage
- All critical paths tested
- Performance benchmarks established
- Load testing completed

#### 8.5.2 Feature Flags
- Implement feature toggles for new changes
- Ability to switch between old/new behavior
- Gradual rollout capability

#### 8.5.3 Blue-Green Deployment
- Maintain both old and new versions
- Quick rollback capability
- Gradual traffic migration

#### 8.5.4 Monitoring and Alerting
- Enhanced monitoring during migration
- Performance metrics tracking
- Error rate monitoring
- Quick issue detection

### 8.6 Immediate Actions (Next Steps)

**Week 1-2:**
1. ✅ Approve this analysis report
2. Form migration team (2-3 developers + QA)
3. Set up parallel development environment
4. Create detailed project plan
5. Establish success criteria

**Week 3-4:**
6. Increase test coverage to ≥70%
7. Set up performance benchmarking
8. Document current system behavior
9. Create rollback procedures
10. Audit all dependencies for Scala 2.13/Pekko compatibility

### 8.7 Success Criteria

**Functional:**
- ✅ All existing features work identically
- ✅ All tests pass
- ✅ No regression in functionality
- ✅ API compatibility maintained

**Performance:**
- ✅ Response time ≤ current baseline + 10%
- ✅ Throughput ≥ current baseline - 5%
- ✅ Resource usage ≤ current + 15%

**Quality:**
- ✅ No critical bugs
- ✅ No P1 issues in first month
- ✅ Code coverage maintained or improved
- ✅ All security scans pass

**Operational:**
- ✅ Deployment successful
- ✅ Rollback procedure tested
- ✅ Monitoring in place
- ✅ Team trained

### 8.8 Alternative: Interim Solution

**If Full Migration Not Feasible Now:**

**Option: Upgrade to Play 2.8.22 + Akka 2.6.21 (Interim)**
- **Timeline:** 4-6 weeks
- **Effort:** Low-Medium
- **Benefits:**
  - Stays Apache 2.0 licensed (Akka 2.6.x)
  - Security updates through 2023
  - Easier migration
  - Minimal breaking changes
- **Drawbacks:**
  - Still EOL (November 2023)
  - Delays inevitable Pekko migration
  - Technical debt continues to grow

**When to Choose Interim:**
- Limited resources currently
- Tight project deadlines
- Need time to prepare team
- Want to validate Play upgrade separately

**Recommendation:** Only if full migration cannot be staffed adequately

---

## 9. Conclusion

### 9.1 Summary

The migration from Play 2.7.2 with Akka 2.5.22 to Play 2.9.x with Apache Pekko is:

✅ **Necessary** - Due to license changes and EOL status
✅ **Feasible** - With proper planning and resources
✅ **Beneficial** - Long-term sustainability and improvements
⚠️ **Complex** - Requires significant effort and testing

### 9.2 Risk Assessment

**Overall Risk Level: MEDIUM-HIGH**

| Risk Factor | Level | Mitigation |
|------------|-------|------------|
| Technical Complexity | High | Phased approach, comprehensive testing |
| Breaking Changes | Medium | Thorough testing, gradual rollout |
| Performance Impact | Medium | Benchmarking, optimization |
| Development Disruption | High | Dedicated team, timeline buffer |
| Operational Risk | Medium | Blue-green deployment, monitoring |
| **Overall** | **Medium-High** | **Comprehensive risk mitigation plan** |

### 9.3 Business Impact

**Positive:**
- ✅ License compliance and legal safety
- ✅ Long-term technical sustainability
- ✅ Improved security posture
- ✅ Modern technology stack
- ✅ Better developer experience

**Negative:**
- ❌ 10-16 weeks development time
- ❌ Resource allocation required
- ❌ Temporary feature freeze
- ❌ Testing overhead
- ❌ Short-term risk of issues

### 9.4 Final Recommendation

**✅ PROCEED WITH MIGRATION**

**Timeline:** Q1 2025 (12-16 weeks)
**Approach:** Phased migration with comprehensive testing
**Target:** Play 2.9.x + Apache Pekko 1.0.x

**Rationale:**
1. Current stack is EOL with no security updates
2. Akka license changes pose legal/business risk
3. Apache Pekko provides sustainable open-source alternative
4. Migration complexity is manageable with proper planning
5. Long-term benefits outweigh short-term costs

**Not recommended:** Staying on current stack beyond 6 months

---

## 10. Appendices

### Appendix A: Detailed File Change List

**Java Files Requiring Import Changes (18 files):**
```
/sb-actor/src/main/java/org/sunbird/actor/core/ActorService.java
/sb-actor/src/main/java/org/sunbird/actor/core/ActorCache.java
/all-actors/src/main/java/org/sunbird/BaseActor.java
/all-actors/src/main/java/org/sunbird/Application.java
/all-actors/src/main/java/org/sunbird/notification/actor/NotificationActor.java
/all-actors/src/main/java/org/sunbird/notification/actor/CreateNotificationActor.java
/all-actors/src/main/java/org/sunbird/notification/actor/ReadNotificationActor.java
/all-actors/src/main/java/org/sunbird/notification/actor/UpdateNotificationActor.java
/all-actors/src/main/java/org/sunbird/notification/actor/DeleteNotificationActor.java
/all-actors/src/main/java/org/sunbird/notification/actor/NotificationTemplateActor.java
/all-actors/src/main/java/org/sunbird/health/actor/HealthActor.java
/service/app/utils/module/SignalHandler.java
/service/app/controllers/BaseController.java
/service/app/controllers/ResponseHandler.java
/all-actors/src/test/java/org/sunbird/notification/actor/BaseActorTest.java
/all-actors/src/test/java/org/sunbird/notification/actor/NotificationActorTest.java
/service/test/controllers/BaseControllerTest.java
/service/test/controllers/DummyActor.java
```

### Appendix B: Configuration Files

**Files Requiring Configuration Changes (2 files):**
```
/service/conf/application.conf
/all-actors/src/main/resources/application.conf
```

### Appendix C: Build Files

**POM files requiring updates (9 files):**
```
/pom.xml (root)
/service/pom.xml
/all-actors/pom.xml
/sb-actor/pom.xml
/sb-utils/pom.xml
/notification-sdk/pom.xml
/cassandra-utils/pom.xml
/sb-telemetry-utils/pom.xml
/sb-common/pom.xml
```

### Appendix D: Reference Links

**Official Documentation:**
- Play Framework: https://www.playframework.com/
- Play 2.9 Migration Guide: https://www.playframework.com/documentation/2.9.x/Migration29
- Apache Pekko: https://pekko.apache.org/
- Pekko Migration Guide: https://pekko.apache.org/docs/pekko/current/project/migration-guides.html
- Akka License Change: https://www.lightbend.com/akka/license-faq

**Community Resources:**
- Pekko GitHub: https://github.com/apache/incubator-pekko
- Play Framework GitHub: https://github.com/playframework/playframework
- Apache Pekko Discuss: https://lists.apache.org/list.html?pekko.apache.org

### Appendix E: Estimated Effort Breakdown

| Task | Duration | Resources | Risk |
|------|----------|-----------|------|
| **Phase 1: Preparation** | | | |
| Analysis & Planning | 1 week | 1 dev | Low |
| Test Suite Enhancement | 1 week | 1 dev + QA | Low |
| **Phase 2: Scala Upgrade** | | | |
| Dependency Updates | 0.5 weeks | 1 dev | Low |
| Compilation & Fixes | 1 week | 2 devs | Medium |
| Testing | 0.5 weeks | QA | Low |
| **Phase 3: Play Upgrade** | | | |
| Dependency Updates | 0.5 weeks | 1 dev | Medium |
| API Migration | 1.5 weeks | 2 devs | Medium |
| Testing | 1 week | QA | Medium |
| **Phase 4: Pekko Migration** | | | |
| Dependency Updates | 0.5 weeks | 1 dev | Low |
| Code Migration | 1.5 weeks | 2 devs | Medium |
| Config Migration | 1 week | 2 devs | High |
| Testing | 1 week | QA | High |
| **Phase 5: Validation** | | | |
| Performance Testing | 1 week | 1 dev + QA | Medium |
| Load Testing | 0.5 weeks | DevOps + QA | Medium |
| Documentation | 0.5 weeks | 1 dev | Low |
| Deployment Testing | 1 week | DevOps | Medium |
| **Total** | **12-16 weeks** | **2-3 devs + QA** | **Medium-High** |

### Appendix F: Dependency Version Matrix

**Current Versions:**
```
Play Framework: 2.7.2
Akka: 2.5.22
Akka HTTP: 10.1.8
Scala: 2.12.11
Java: 11
```

**Target Versions (Recommended):**
```
Play Framework: 2.9.5
Apache Pekko: 1.0.3
Pekko HTTP: 1.0.1
Scala: 2.13.14
Java: 11 (or 17)
```

**Intermediate Option (If needed):**
```
Play Framework: 2.8.22
Akka: 2.6.21 (Last Apache 2.0)
Akka HTTP: 10.2.10
Scala: 2.12.11 or 2.13.x
Java: 11
```

---

## Document Information

**Version:** 1.0
**Date:** October 7, 2025
**Status:** Draft for Review
**Author:** Technical Analysis Team
**Repository:** https://github.com/SNT01/notification-service

**Next Review Date:** After stakeholder feedback

**Approval Required From:**
- Technical Lead
- Product Owner
- DevOps Team
- QA Team Lead

---

## Glossary

- **Akka:** Actor-based toolkit for building concurrent applications (formerly Apache 2.0, now BSL)
- **Apache Pekko:** Apache Software Foundation fork of Akka 2.6.x (Apache 2.0 licensed)
- **BSL:** Business Source License - source-available but not open-source license
- **EOL:** End of Life - no longer supported or receiving updates
- **Play Framework:** Web framework for Scala and Java
- **SBT:** Scala Build Tool - the standard build tool for Scala projects
- **Artery:** High-performance remoting implementation in Pekko (successor to Netty-based remoting)

---

**END OF REPORT**
