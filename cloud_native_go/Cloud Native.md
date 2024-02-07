- Scalability
	- Vertical
	- Horizontal (State management is key)

- Loose Coupling
	- Avoid distributed monoliths

- Resilience 
	- Design for failure
	- The ability to recover from or handle failure
	- Not = reliability (ability to function as expected for a given time)

- Manageability
	- Alter system behaviour without changing the code

- Observability 
	- Logs, metrics, tracing

# Dependability
![[Pasted image 20240207080316.png]]

![[Pasted image 20240207080729.png]]

## Fault Prevention
#### Programming Practices 
- TDD
- Reviews
- Linting
- CI
#### Language Features

#### Scalability

#### Loose Coupling

## Fault Tolerance

Error detection and recovery

## Fault Removal

#### Verification and testing
- Static Analysis - Scanners
- Dynamic Analysis - Testing

#### Manageability

## Fault Forecasting



# Scalability
## Four common bottlenecks
- CPU
- Memory
- Disk I/O
- Network I/O

## State
### Application vs. Resource State
- Application (e.g. session state) is the hard one

