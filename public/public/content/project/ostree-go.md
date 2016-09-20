+++
client_name=""
date = "2016-07-25"
image = ""
image_preview = ""
summary = "Golang bindings for ostree, a \"git-like\" model for committing filesystem trees"
tags = ["golang", "c", "open-source"]
title = "Ostree-Go"
external_link = "github.com/ostreedev/ostree-go"

+++

OSTree is a tool that combines a "git-like" model for committing and downloading bootable fiesystem trees, along with a layer for deploying them and managing the bootloader configuration.  It was designed so that package managers, system upgrade tools, etc. can use OSTree as a "deduplicating hardlink store."  Recently, OSTree has become popular for use with containers.  However, many platforms that work with container do so in Go.  This library provides a set of Go bindings for OSTree that mimic the command line as much as possible so as it make it easy for container teams to use OSTree.

