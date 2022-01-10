---
hide: false
title: Some notes about BASH programming
toc: false
comments: true
layout: post
description: A post to keep track of some BASH programming construct
categories: [Bash, DevOps, Linux]
---

This page will serve as a reference for my perigrination with BASH. A good place to keep track of BASH constructs that I met during my personal and professional projects.

## A basic command line interface

To setup a command line interface for your bash scripts, you can follow this code snippet:

```bash
THISSCRIPT=$(basename $0)
VAR=""

usage() {
    echo ""
    echo "Usage: $THISSCRIPT [-h|--help]"
    echo ""
    echo "  Describe here what this script will do"
    echo ""
    echo "Params:"
    echo "        -a1|--argument1: description ... "
    echo "        -a2|--argument2: description ... "
    echo "        -an|--argumentn: description ... "
    echo "         -h|--help: description ... "
    echo ""
    echo "Examples:"
    echo ""
    echo "        ./$THISSCRIPT"    
}

# Parsing arguments

while [[ $# -gt 0 ]]
do
    case $1 in
        -a1|--argument1)
        shift
        echo "Hello a1" # put here any logic or code execution you want
        ;;
        -a2|--argument2)
        shift
        VAR=$1 # you can assign a value to a variable initialized before
        ;;
        -an|--argumentn)
        shift
        echo "Hello an" # put here any logic or code execution you want
        ;;
        -h|--help)
        usage
        exit 0 # use the exit command if you want the script to exit after an action
        ;;
        *)
        echo >$2 "$THISSCRIPT Invalid argument: $1"
        usage
        exit 1
        ;;
    esac
    shift
done

# you need to validate VAR
[[ -z "$VAR" ]] && { echo >$2 "$THISSCRIPT ERROR: please specify a value for VAR"; exit 1; }
```

### Code explanation

* To get the name of the script file  
  
  ```bash
  THISSCRIPT=$(basename $0)
  ```

* `while` loop
  
  ```bash
  while [[ condition ]]
  do ...
  done
  ```

* `case` statement: 
  
  ```bash
  case $1 in
      -a1|--argument1)
   shift
   # do something
   ;;
  esac
  ```
  
  > `esac` is the keyword used to end a case statement

* The `[[ -z "$string" ]]` condition return `True` if string is empty

* Redirect STDOUT to STDERROR
  
  ```bash
  echo >$2 "your text"
  ```
