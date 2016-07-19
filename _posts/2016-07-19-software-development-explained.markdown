Metaphor of all software development: I keep trying to add features to a project, only
to uncover bugs that need to be solved before I can go on :D

Some of them are very interesting, and once I manage to find the root cause they will
likely warrant their own posts.

However, in better news, [https://github.com/calamares/calamares](Calamares) now has
support for installing groups of packages during the system installation. It will
be part of the new release. This was added by @AlmAck and I so that we would be able
to ship lightweight, minimal Chakra ISOs with only a small set of required packages
installed. The users are then able to choose groups of packages to install from the
live directly, and have them there once they reboot into the new system. This feature
was available in our old installer, Tribe, so we are happy to provide it again starting
from the next Chakra release.

Since this needs to work for every distribution using Calamares, both the list of
packages and the package manager to use for installation are configurable in their
respective modules. The README file inside the *netinstall* module should contain
all info required to deploy this in any Calamares build.
