Custom paths for the visual studio projects are defined in paths.targets.  

All projects import this targets file, and it should be in the same directory as the project.

The custom paths are:

OpenSSH-Src-Path            =  The directory path of the OpenSSH root source directory (with trailing slash)
OpenSSH-Bin-Path            =  The directory path of the location to which binaries are placed.  This is the output of the binary projects
OpenSSH-Lib-Path            =  The directory path of the location to which libraries are placed.  This is the output of the libary projects
LibreSSL-x86-Path           =  The directory path of LibreSSL statically compiled for x86 platform.
LibreSSL-x64-Path           =  The directory path of LibreSSL statically compiled for x64 platform.

Notes on FIDO2 support
----------------------

* How to build:

  - Open Windows Terminal.

  - Check out https://github.com/yubico/libfido2 HEAD:

    $ git clone https://github.com/yubico/libfido2

  - Run windows\build.ps1:

    $ cd libfido2\windows
    $ .\build.ps1
  
  - Copy the content of libfido2\output\pkg to openssh-portable\contrib\win32\openssh\libfido2:

    $ Copy-Item -Path libfido2\output\pkg -Destination openssh-portable\contrib\win32\openssh\libfido2 -Recurse

  - Copy webauthn.lib to openssh-portable\contrib\win32\openssh\libfido2\Win64\Release\v142\static:

    $ Copy-Item -Path 'C:\Program Files (x86)\Windows Kits\10\Lib\10.0.20348.0\um\x64\webauthn.lib' -Destination openssh-portable\contrib\win32\openssh\libfido2\Win64\Release\v142\static

  - Set the VS140COMNTOOLS environment variable:

    $ $Env:VS140COMNTOOLS = 'C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\Tools\'

  - Create an empty workspace directory:

    $ mkdir workspace
    $ $Env:WORKSPACE = 'path\to\workspace'

  - Build OpenSSH for Windows:

    $ cd \path\to\openssh-portable\..
    $ .\openssh-portable\contrib\win32\openssh\OpenSSH-build.ps1

* What has been tested:

  * Using a Yubico Security Key supporting FIDO2.1 Preview, with a FIDO2 PIN set.

  - Set the SSH_SK_HELPER environment variable:

    $ $Env:SSH_SK_HELPER = '\path\to\openssh-portable\bin\x64\Release\ssh-sk-helper.exe'

  - Create a new SSH key:

    $ \path\to\openssh-portable\bin\x64\Release\ssh-keygen.exe -t ed25519-sk -O resident -O verify-required

    * Save the key material in SSH's default paths without an associated passphrase.

  - Add the SSH key to your GitHub account.

  - Tell git to use our SSH build:

    $ $Env:GIT_SSH = '\path\to\openssh-portable\bin\x64\Release\ssh.exe'

  - Clone a repository using the SSH key for authentication:

    $ git clone ssh://git@github.com/org/some-private-repo

* What hasn't been tested, but should work:

  * the ecdsa-sk key type;
  * ssh-keygen without -O resident -O verify-required;
  * ssh-keygen with either -O resident or -O verify-required.

* What definitely doesn't work:

  * ssh-keygen -O no-touch-required:
    - there does not appear to be a way to toggle user presence in WEBAUTHN_AUTHENTICATOR_GET_ASSERTION_OPTIONS.
  * ssh-keygen -K, ssh-add -K:
    - these use Credential Management to reconstruct the SSH private key.
