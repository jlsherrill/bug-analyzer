# Bug Solver Repositories

This file tells the bug analyzer which repositories to search when analyzing Jira issues.

The user will interact with image-builder in many different ways
- Through a web interface hosted on console.redhat.com (image-builder-frontend)
- Through a web interface hosted locally and running in cockpit (image-builder-frontend).  These two frontends behave differently
- Through the new image-builder cli
- Through the much older osbuild-composer cli 
- The hosted api on console.redhat.com (image-builder-crc).  This talks to composer to actually build the images.


## ./repos/image-builder-cli
**Repository:** https://github.com/osbuild/image-builder-cli.git  
**Purpose:** Command-line tool for building operating system images in a convenient way
**Stack:** Go, osbuild, podman/containers
**Covers:** CLI commands, image building, blueprint configurations, SBOM generation, cloud uploads (AWS), cross-architecture builds
**Key areas:**
- cmd/image-builder/ - Main CLI binary
- Build commands and options
- Blueprint/config parsing (TOML/JSON)
- Cloud integration (AWS AMI uploads)
- Registration/subscription handling
- Repository configuration

## ./repos/image-builder-crc
**Repository:** https://github.com/osbuild/image-builder-crc.git  
**Purpose:** HTTP API middleware serving as backend for Image Builder on console.redhat.com
**Stack:** Go, FastAPI-style HTTP services, OpenAPI
**Covers:** REST API endpoints, osbuild-composer integration, console.redhat.com backend services
**Key areas:**
- internal/v1/ - API v1 implementation
- internal/v1/api.yaml - OpenAPI specification
- HTTP API handlers
- Osbuild composer integration
- Authentication and authorization

## ./repos/image-builder-frontend
**Repository:** https://github.com/osbuild/image-builder-frontend.git  
**Purpose:** Customer-facing web application for Image Builder on console.redhat.com, also available as Cockpit plugin
**Stack:** React 18, TypeScript, Redux Toolkit, RTK Query, PatternFly 6, Webpack, Vitest, Playwright
**Covers:** UI bugs, wizard flows, authentication, image creation workflows, cloud provider integrations, Cockpit plugin
**Key areas:**
- src/Components/ - React components organized by feature
- src/store/slices/ - Redux store and RTK Query API definitions
- src/Utilities/ - Shared utility functions
- src/Hooks/ - Custom React hooks
- playwright/ - E2E tests
- cockpit/ - Cockpit plugin build configuration
- api/config/ - RTK Query codegen configuration

## ./repos/images
**Repository:** https://github.com/osbuild/images  
**Purpose:** Go library for generating osbuild manifests and uploading artifacts
**Stack:** Go, YAML-based image definitions
**Covers:** Manifest generation, cloud platform uploads (AWS, Azure, GCP), Koji uploads, distro definitions
**Key areas:**
- pkg/ - Core library packages
- pkg/container/ - Container-related functionality
- pkg/upload/ - Cloud and Koji upload implementations
- data/distrodefs/ - YAML-based distribution definitions
- data/repositories/ - Distribution repository configurations
- cmd/gen-manifests/ - Manifest generation tool
- cmd/build/ - Build tool

## ./repos/osbuild
**Repository:** https://github.com/osbuild/osbuild.git  
**Purpose:** Pipeline-based build system for operating system artifacts and images
**Stack:** Python 3.6+, bubblewrap, bash, systemd
**Covers:** Pipeline execution engine, stages/sources/inputs/mounts/devices, manifest processing, build isolation
**Key areas:**
- stages/ - OSBuild pipeline stages (never broken, only deprecated)
- osbuild/ - Core Python module
- sources/ - Data source definitions
- assemblers/ - Image assembly logic
- test/data/manifests/ - Test manifests
- Manifest schema definitions

## ./repos/osbuild-composer
**Repository:** https://github.com/osbuild/osbuild-composer.git  
**Purpose:** HTTP services for composing operating system images, orchestrator for image builds
**Stack:** Go, systemd services, Lorax-composer API compatibility
**Covers:** Image composition orchestration, blueprint processing, cloud API, hosted service backend, on-premises deployment
**Key areas:**
- internal/cloudapi/ - Cloud API implementation (never broken)
- internal/blueprint/ - Blueprint parsing and processing
- cmd/osbuild-composer/ - Main composer service
- HTTP service handlers
- Build orchestration logic
- Target upload implementations
- Lorax-composer API compatibility layer

## ./repos/bootc-image-builder
**Repository:** https://github.com/osbuild/bootc-image-builder.git  
**Purpose:** Container to create disk images from bootc container inputs, uses the images library under the hood
**Stack:** Container-based, osbuild, podman/containers, Go
**Covers:** Disk image creation from bootc containers, multiple output formats (QCOW2, AMI, ISO, VMDK, RAW, VHD, GCE), cloud uploads (AWS), build configurations, Anaconda installer ISOs
**Key areas:**
- bib/ - Main builder implementation
- Containerfile - Container build definition
- Build config support (TOML/JSON blueprints)
- Image types (ami, qcow2, vmdk, bootc-installer, anaconda-iso, raw, vhd, gce, pxe-tar-xz)
- Cloud uploaders (AWS S3, AMI import)
- Filesystem and partition customizations
- User and kernel argument customizations
