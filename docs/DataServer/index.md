# Data Server

## Introduction

This microservice is a rest API-based system. The DataServer's role is to provide essential data navigating essential data from the system.

Data Server shares data with the client to provide video data, video manifest, and other data sets from the system that can be used for analytical purposes.

The Data server also performs tokenization of data to ensure secured access resources. This feature is essential for accessing video chunks via FSS(File Store System). FSS has mandatory use of `file_token`, for ensuring the safety of private data as an additional layer of security.

