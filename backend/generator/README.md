# Generator service

## Overview

Generator get parameteric question or exam and process it to generate the result question or exam.

> **Generator** is a GRPC service for internal use.

It will use the already existing Generator library to generate the result question or exam and cache the result for future use.

## Cache

- The generator will cache the generated result question or exam for future use.
- It produces unique hash of the generated question or exam to identify it later when evaluating the result.