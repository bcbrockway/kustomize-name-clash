We have a multi-layered kustomize config laid out as follows (actually they're in different paths but this should illustrate the issue we have well enough):

TREE HERE

We're trying to reduce the amount of double-keying as much as possible by having an elasticsearch StatefulSet in its own base and then deriving master and data nodes from it, but we've run into an issue when using variables. This is the base StatefulSet:

