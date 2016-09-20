+++
client_name = ""
date = "2016-08-12"
image = ""
image_preview = ""
summary = "Watch for images being pushed onto an OpenShift registry and explode them on to persistent storage"
tags = ["golang", "docker", "containers"]
title = "Os-explode"

+++

Enterprises that run containers on their public, private, and hybrid clouds want to know that those images are secure.  Os-explode is a tool that sets of the containers to be scanned by a scanning container such as the OpenSCAP container.  Os-explode watches an OpenShift image stream for images being pushed up to a local registry.  It then takes the layers from the images stream and commits each layer into an OSTree repository.  These committed images can then be scanned by scanning containers manually, automatically (i.e. when committed) or on a schedule.  The container images can also be run using runc, a tool for spawning and running containers according to the OCI specification.
