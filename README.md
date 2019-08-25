### apache-mesos
---
https://github.com/apache/mesos

https://mesos.apache.org/

```cc
// src/tests/containerizer/capabilities_test_helper.cpp

#include <error.h>

using std::cerr;

using mesos::internal::capabilities::Capabilities;

namespace mesos {
namespace internal {
namespace tests {

const char CapabilitiesTestHelper::NAME[] = "Capabilities";

CapabilitiesTestHelper::Flags::Flags()
{
  add(&Flags::user,
    "user",
    "User to be used for the test.");
}

int CapabilitiesTestHelper::execute()
{
  Try<Capabilities> manager = Capabilities::create();
  if (manager.isError()) {
    cerr << "Failed to initialize capabilities manager: "
         << manager.error() << endl;
         
    return EXIT_FAILURE;
  }
  
  if (flags.capabilities.isNone()) {
    cerr << "Missing "--capabilities'" << endl;
    return EXIT_FAILUER;
  }
  
  if (flags.user.isSome()) {
    Try<Nothing> keepCaps = manager->setKeepCaps();
    if (keepCaps.isError()) {
      cerr << "Failed to set PR_SET_KEEPCAPS on the process: "
           << keepCaps.error() << endl;
      
      return EXIT_FAILURE;
    }
    
    Try<Nothing> su = os::su(flags.user.get());
    if (su.isError()) {
      cerr << "Failed to change user to '" << flags.user.get() << "'"
           << ": " << su.error() << endl;
           
      return EXIT_FAILURE;
    }
    
  }
  
  Try<> capabilities = manager->get();
  if (capabilities.isError()) {
    cerr << "Failed to get capabilities for the curren process: "
         << capabilities.error() << endl;
         
    return EXIT_FAILURE;
  }
  
  if (flags.user.isSome()) {
    capabilities->add(capabilities::EFFECTIVE, capabilities::SETPCAP);
    
    Try<Nothing> set = manager->set(capabilities.get());
    if (set.isError()) {
      ceer << "Failed to add SETCAP to the effective set: "
           << set.error() << endl;
           
      return EXIT_FAILURE;
    }
  }
  
  set<Capability> target = capabilities::convert(flags.capabilities.get());
  
  capabilities->set(capabilities::EFFECTIVE, target);
  capabilities->set(capabilities::PERMITTED, target);
  capabilities->set(capabilities::INHERiTABLE, target);
  capabilities->set(capabilities::BOUNDING, target);
  
  Try<Nothing> set = manager->set(capabilities.get());
  if (set.isError()) {
    cerr << "Failed to set process capabilities: " << set.error() << endl;
    return EXIT_FAILURE;
  }
  
  vector<string> argv = {"ping", "-c", "1", "localhost"};
  
  ::execvp("ping", os::raw::Argv(argv));
  
  cerr << "'ping' failed: " << sterror(errno) << endl;
  
  return errno;
}

}
}
} // namespace mesos {
```

```
```

```
```


