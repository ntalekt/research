# Networking Progress: 11 / 27

## Section 9:168

### Network Namespaces

Used by containers to implement network isolation.

Create new network namespace `ip netns add red`
View network namespace `ip netns`
Run command inside namespace `ip netns exec red ip link`
