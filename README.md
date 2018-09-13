# FMI4cpp (work in progress)

FMI4cpp is a cross-platform FMI 2.0 implementation written in modern C++.

Looking for an easy to install, easy to use, easy to reason with, object oriented and fast FMI implementation for C++?    
You are in luck. FMI4cpp is all of those.

### Why should I use this over other C/C++ FMI Libraries

Because this library has a much cleaner API and is significantly easier to use.


### How do I use it in my own project?

When the library is stable I will add it to vcpkg.


### Development dependencies

Install [vcpkg](https://github.com/Microsoft/vcpkg) and run:

```
./vcpkg install boost-property-tree boost-filesystem libzip
``` 

On linux you _might_ also need to install some additional libraries:

```
./vcpkg install zlib bzip2 openssl
``` 

These can probably also be installed through the native package handler. 

#### API

```cpp

#include <iostream>
#include <fmicpp/fmicpp.hpp>

using namespace fmicpp::fmi2;

const double stop = 10.0;
const double stepSize = 1.0/100;

int main() {

    import::Fmu fmu("path/to/fmu.fmu");
    
    auto md = fmu.getModelDescription();
    xml::ScalarVariable var = md.getVariableByName("my_var");
    
    auto md_cs = md.asCoSimulationModelDescription();
    std::cout << "modelIdentifier=" << md_cs->modelIdentifier() << std::endl;
    
    auto slave = fmu.asCoSimulationFmu().newInstance();
    slave->init();
   
    double t;
    fmi2Real value;
    fmi2Status status;
    fmi2ValueReference vr = var.getValueReference();
    while ( (t = slave->getSimulationTime()) <= stop) {
    
        status = slave->doStep(stepSize);
        if (status != fmi2OK) {
            //error handling
            break;
        }
        
        status = slave->readReal(vr, value);
        if (status != fmi2OK) {
            //error handling
            break;
        }
        std::cout << "t=" << t << ", " << var.getName() << "=" << value << std::endl;
     
    }
    
    slave->terminate();
    
}
```

***

#### Running examples/tests

In order to run the example/test code, a system variable named __TEST_FMUs__ must be present on your system. 
This variable should point to the location of the content found [here](https://github.com/markaren/TEST_FMUs).

To build the examples pass ```-DBUILD_EXAMPLES``` to CMmake.
Similarly, to build the tests, pass ```-DBUILD_TESTS``` to CMake.
