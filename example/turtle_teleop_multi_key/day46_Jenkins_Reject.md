# ROS 패키지 배포 실패
이전에 릴리즈 했던 `turtle_teleop_multi_key` 패키지가 `rosdistro`에는 성공적으로 `merge` 되었지만, 젠킨스 배포에서 실패해 다음과 같은 내용이 메일로 왔습니다.

```
See <http://build.ros.org/job/Mdoc__turtle_teleop_multi_key__ubuntu_bionic_amd64/1/display/redirect>

Changes:


------------------------------------------
[...truncated 29.59 KB...]
bionic: Pulling from library/ubuntu
Digest: sha256:a61728f6128fb4a7a20efaa7597607ed6e69973ee9b9123e3b4fd28b7bba100b
Status: Image is up to date for ubuntu:bionic
docker.io/library/ubuntu:bionic
+ docker build --force-rm -t doc_task_generation.melodic_turtle_teleop_multi_key .
Sending build context to Docker daemon  15.87kB
Step 1/26 : FROM ubuntu:bionic
---> 2eb2d388e1a2
Step 2/26 : VOLUME ["/var/cache/apt/archives"]
---> Using cache
---> a1774f04c62b
Step 3/26 : ENV DEBIAN_FRONTEND noninteractive
---> Using cache
---> 76353b5e1eb5
Step 4/26 : RUN for i in 1 2 3; do apt-get update && apt-get install -q -y locales && apt-get clean && break || if [ $i -lt 3 ]; then sleep 5; else false; fi; done
---> Using cache
---> bcdfe6e29c5c
Step 5/26 : RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
---> Using cache
---> d02f2fdfd81e
Step 6/26 : RUN locale-gen en_US.UTF-8
---> Using cache
---> 05334829c391
Step 7/26 : ENV LANG en_US.UTF-8
---> Using cache
---> 8217b59ce88b
Step 8/26 : ENV TZ UTC+00
---> Using cache
---> e0be95a1e111
Step 9/26 : RUN useradd -u 1001 -l -m buildfarm
---> Using cache
---> 4090d3f744fb
Step 10/26 : RUN mkdir /tmp/keys
---> Using cache
---> ca6be19246ed
Step 11/26 : RUN for i in 1 2 3; do apt-get update && apt-get install -q -y gnupg && apt-get clean && break || if [ $i -lt 3 ]; then sleep 5; else false; fi; done
---> Using cache
---> 873f1d6449fd
Step 12/26 : RUN echo "-----BEGIN PGP PUBLIC KEY BLOCK-----\nVersion: GnuPG v1\n\nmQINBFzvJpYBEADY8l1YvO7iYW5gUESyzsTGnMvVUmlV3XarBaJz9bGRmgPXh7jc\nVFrQhE0L/HV7LOfoLI9H2GWYyHBqN5ERBlcA8XxG3ZvX7t9nAZPQT2Xxe3GT3tro\nu5oCR+SyHN9xPnUwDuqUSvJ2eqMYb9B/Hph3OmtjG30jSNq9kOF5bBTk1hOTGPH4\nK/AY0jzT6OpHfXU6ytlFsI47ZKsnTUhipGsKucQ1CXlyirndZ3V3k70YaooZ55rG\naIoAWlx2H0J7sAHmqS29N9jV9mo135d+d+TdLBXI0PXtiHzE9IPaX+ctdSUrPnp+\nTwR99lxglpIG6hLuvOMAaxiqFBB/Jf3XJ8OBakfS6nHrWH2WqQxRbiITl0irkQoz\npwNEF2Bv0+Jvs1UFEdVGz5a8xexQHst/RmKrtHLct3iOCvBNqoAQRbvWvBhPjO/p\nV5cYeUljZ5wpHyFkaEViClaVWqa6PIsyLqmyjsruPCWlURLsQoQxABcL8bwxX7UT\nhM6CtH6tGlYZ85RIzRifIm2oudzV5l+8oRgFr9yVcwyOFT6JCioqkwldW52P1pk/\n/SnuexC6LYqqDuHUs5NnokzzpfS6QaWfTY5P5tz4KHJfsjDIktly3mKVfY0fSPVV\nokdGpcUzvz2hq1fqjxB6MlB/1vtk0bImfcsoxBmF7H+4E9ZN1sX/tSb0KQARAQAB\ntCZPcGVuIFJvYm90aWNzIDxpbmZvQG9zcmZvdW5kYXRpb24ub3JnPokCVAQTAQoA\nPhYhBMHPbjHmut6IaLFytPQu1vurF8ZUBQJc7yaWAhsDBQkDwmcABQsJCAcCBhUK\nCQgLAgQWAgMBAh4BAheAAAoJEPQu1vurF8ZUkhIP/RbZY1ErvCEUy8iLJm9aSpLQ\nnDZl5xILOxyZlzpg+Ml5bb0EkQDr92foCgcvLeANKARNCaGLyNIWkuyDovPV0xZJ\nrEy0kgBrDNb3++NmdI/+GA92pkedMXXioQvqdsxUagXAIB/sNGByJEhs37F05AnF\nvZbjUhceq3xTlvAMcrBWrgB4NwBivZY6IgLvl/CRQpVYwANShIQdbvHvZSxRonWh\nNXr6v/Wcf8rsp7g2VqJ2N2AcWT84aa9BLQ3Oe/SgrNx4QEhA1y7rc3oaqPVu5ZXO\nK+4O14JrpbEZ3Xs9YEjrcOuEDEpYktA8qqUDTdFyZrxb9S6BquUKrA6jZgT913kj\nJ4e7YAZobC4rH0w4u0PrqDgYOkXA9Mo7L601/7ZaDJob80UcK+Z12ZSw73IgBix6\nDiJVfXuWkk5PM2zsFn6UOQXUNlZlDAOj5NC01V0fJ8P0v6GO9YOSSQx0j5UtkUbR\nfp/4W7uCPFvwAatWEHJhlM3sQNiMNStJFegr56xQu1a/cbJH7GdbseMhG/f0BaKQ\nqXCI3ffB5y5AOLc9Hw7PYiTFQsuY1ePRhE+J9mejgWRZxkjAH/FlAubqXkDgterC\nh+sLkzGf+my2IbsMCuc+3aeNMJ5Ej/vlXefCH/MpPWAHCqpQhe2DET/jRSaM53US\nAHNx8kw4MPUkxExgI7Sd\n=4Ofr\n-----END PGP PUBLIC KEY BLOCK-----" > /tmp/keys/0.key && apt-key add /tmp/keys/0.key
---> Using cache
---> d9f923f24274
Step 13/26 : RUN echo deb http://repositories.ros.org/ubuntu/testing bionic main | tee -a /etc/apt/sources.list.d/buildfarm.list
---> Using cache
---> cfcbec443ecf
Step 14/26 : RUN mkdir /tmp/wrapper_scripts
---> Using cache
---> 95d61faa8d79
Step 15/26 : RUN echo "#!/usr/bin/env python3\n\n# Copyright 2014-2016 Open Source Robotics Foundation, Inc.\n#\n# Licensed under the Apache License, Version 2.0 (the \"License\");\n# you may not use this file except in compliance with the License.\n# You may obtain a copy of the License at\n#\n#     http://www.apache.org/licenses/LICENSE-2.0\n#\n# Unless required by applicable law or agreed to in writing, software\n# distributed under the License is distributed on an \"AS IS\" BASIS,\n# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.\n# See the License for the specific language governing permissions and\n# limitations under the License.\n\nimport subprocess\nimport sys\nfrom time import sleep\n\n\ndef main(argv=sys.argv[1:]):\n    max_tries = 10\n    known_error_strings = [\n        'Failed to fetch',\n        'Failed to stat',\n        'Hash Sum mismatch',\n        'Unable to locate package',\n        'is not what the server reported',\n    ]\n\n    command = argv[0]\n    if command in ['update', 'source']:\n        rc, _, _ = call_apt_repeatedly(\n            argv, known_error_strings, max_tries)\n        return rc\n    elif command == 'update-install-clean':\n        return call_apt_update_install_clean(\n            argv[1:], known_error_strings, max_tries)\n    else:\n        assert \"Command '%s' not implemented\" % command\n\n\ndef call_apt_update_install_clean(\n        install_argv, known_error_strings, max_tries):\n    tries = 0\n    command = 'update'\n    while tries < max_tries:\n        if command == 'update':\n            rc, _, tries = call_apt_repeatedly(\n                [command], known_error_strings, max_tries - tries,\n                offset=tries)\n            if rc != 0:\n                # abort if update was unsuccessful even after retries\n                break\n            # move on to the install command if update was successful\n            command = 'install'\n\n        if command == 'install':\n            # any call is considered a try\n            tries += 1\n            known_error_strings_redo_update = [\n                'Size mismatch',\n                'maybe run apt update',\n                'The following packages cannot be authenticated!',\n                'Unable to locate package',\n                'has no installation candidate',\n                'corrupted package archive',\n            ]\n            rc, known_error_conditions = \\\\\n call_apt(\n                    [command] + install_argv,\n                    known_error_strings + known_error_strings_redo_update)\n            if not known_error_conditions:\n                if rc != 0:\n                    # abort if install was unsuccessful\n                    break\n                # move on to the clean command if install was successful\n                command = 'clean'\n                continue\n\n            # known errors are always interpreted as a non-zero rc\n            if rc == 0:\n                rc = 1\n            # check if update needs to be rerun\n            if (\n                set(known_error_conditions) &\n                set(known_error_strings_redo_update)\n            ):\n                command = 'update'\n                print(\"'apt install' failed and likely requires \" +\n                      \"'apt update' to run again\")\n         # retry with update command\n                continue\n\n            print('')\n            print('Invocation failed due to the following known error '\n                  'conditions: ' + ', '.join(known_error_conditions))\n            print('')\n            if tries < max_tries:\n                sleep_time = 5\n                print(\"Reinvoke 'apt install' after sleeping %s seconds\" %\n         sleep_time)\n                sleep(sleep_time)\n                # retry install command\n\n        if command == 'clean':\n           rc, _ = call_apt([command], [])\n            break\n\n    return rc\n\n\ndef call_apt_repeatedly(argv, known_error_strings, max_tries, offset=0):\n    command = argv[0]\n    for i in range(1, max_tries + 1):\n        if i > 1:\n            sleep_time = 5 + 2 * (i + offset)\n            print(\"Reinvoke 'apt %s' (%d/%d) after sleeping %s seconds\" %\n                  (command, i + offset, max_tries + offset, sleep_time))\n            sleep(sleep_time)\n        rc, known_error_conditions = call_apt(argv, known_error_strings)\n        if not known_error_conditions:\n            # break the loop and return the reported rc\n            break\n        # known errors are always interpreted as a non-zero rc\n        if rc == 0:\n            rc = 1\n        print('')\n        print('Invocation failed due to the following known error conditions: '\n              ', '.join(known_error_conditions))\n        print('')\n        # retry in case of failure with known error condition\n    return rc, known_error_conditions, i + offset\n\n\ndef call_apt(argv, known_error_strings):\n    known_error_conditions = []\n\n   # some of the used options are not supported in older distros\n    # e.g. Ubuntu Wily, Debian Jessie\n    cmd = ['apt-get'] + argv\n    print(\"Invoking '%s'\" % ' '.join(cmd))\n    proc = subprocess.Popen(\n        cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)\n    lines = []\n    while True:\n        line = proc.stdout.readline()\n        if not line:\n            break\n        line = line.decode()\n        lines.append(line)\n        sys.stdout.write(line)\n        for known_error_string in known_error_strings:\n            if known_error_string in line:\n     if known_error_string not in known_error_conditions:\n                    known_error_conditions.append(known_error_string)\n    proc.wait()\n    rc = proc.returncode\n    if rc and not known_error_conditions:\n        print('Invocation failed without any known error condition, '\n              'printing all lines to debug known error detection:')\n        for index, line in enumerate(lines):\n            print(' ', index + 1, \"'%s'\" % line.rstrip('\\\\n\\\\r'))\n        print('None of the following known errors were detected:')\n        for index, known_error_string in enumerate(known_error_strings):\n            print(' ', index + 1, \"'%s'\" % known_error_string)\n return rc, known_error_conditions\n\n\nif __name__ == '__main__':\n    sys.exit(main())" > /tmp/wrapper_scripts/apt.py
---> Using cache
---> 4f6a210c0e91
Step 16/26 : RUN echo "#!/usr/bin/env python3\n\n# Copyright 2016 Open Source Robotics Foundation, Inc.\n#\n# Licensed under the Apache License, Version 2.0 (the \"License\");\n# you may not use this file except in compliance with the License.\n# You may obtain a copy of the License at\n#\n#     http://www.apache.org/licenses/LICENSE-2.0\n#\n# Unless required by applicable law or agreed to in writing, software\n# distributed under the License is distributed on an \"AS IS\" BASIS,\n# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.\n# See the License for the specific language governing permissions and\n# limitations under the License.\n\nimport subprocess\nimport sys\nfrom time import sleep\n\n\ndef main(argv=sys.argv[1:]):\n    max_tries = 10\n    known_error_strings = [\n        'Connection timed out',\n    ]\n\n    command = argv[0]\n    if command == 'clone':\n        rc, _, _ = call_git_repeatedly(\n            argv, known_error_strings, max_tries)\n        return rc\n    else:\n        assert \"Command '%s' not implemented\" % command\n\n\ndef call_git_repeatedly(argv, known_error_strings, max_tries):\n    command = argv[0]\n    for i in range(1, max_tries + 1):\n        if i > 1:\n            sleep_time = 5 + 2 * i\n            print(\"Reinvoke 'git %s' (%d/%d) after sleeping %s seconds\" %\n                  (command, i, max_tries, sleep_time))\n            sleep(sleep_time)\n        rc, known_error_conditions = call_git(argv, known_error_strings)\n        if rc == 0 or not known_error_conditions:\n            break\n        print('')\n        print('Invocation failed due to the following known error conditions: '\n              ', '.join(known_error_conditions))\n       print('')\n        # retry in case of failure with known error condition\n    return rc, known_error_conditions, i\n\n\ndef call_git(argv, known_error_strings):\n    known_error_conditions = []\n\n    cmd = ['git'] + argv\n    print(\"Invoking '%s'\" % ' '.join(cmd))\n    proc = subprocess.Popen(\n        cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)\n    while True:\n        line = proc.stdout.readline()\n        if not line:\n            break\n       line = line.decode()\n        sys.stdout.write(line)\n        for known_error_string in known_error_strings:\n            if known_error_string in line:\n                if known_error_string not in known_error_conditions:\n                    known_error_conditions.append(known_error_string)\n    proc.wait()\n    rc = proc.returncode\n    return rc, known_error_conditions\n\n\nif __name__ == '__main__':\n    sys.exit(main())" > /tmp/wrapper_scripts/git.py
---> Using cache
---> e91b352a5a54
Step 17/26 : RUN echo "2020-08-06 (+0000)"
---> Using cache
---> 16eead899d73
Step 18/26 : RUN for i in 1 2 3; do apt-get update && apt-get install -q -y python3 && apt-get clean && break || if [ $i -lt 3 ]; then sleep 5; else false; fi; done
---> Using cache
---> 66b58e7ebf15
Step 19/26 : RUN python3 -u /tmp/wrapper_scripts/apt.py update-install-clean -q -y git mercurial python3-apt python3-catkin-pkg-modules python3-empy python3-rosdep python3-rosdistro-modules subversion wget
---> Running in 4ccbd94a5cf8
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Get:5 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Fetched 88.7 kB in 0s (235 kB/s)
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (2/10) after sleeping 9 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:3 http://repositories.ros.org/ubuntu/testing bionic InRelease
Err:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (3/10) after sleeping 11 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (4/10) after sleeping 13 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (5/10) after sleeping 15 seconds
Invoking 'apt-get update'
Hit:1 http://repositories.ros.org/ubuntu/testing bionic InRelease
Err:2 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (6/10) after sleeping 17 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:2 http://repositories.ros.org/ubuntu/testing bionic InRelease
Err:3 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (7/10) after sleeping 19 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (8/10) after sleeping 21 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (9/10) after sleeping 23 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:3 http://repositories.ros.org/ubuntu/testing bionic InRelease
Err:4 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Reinvoke 'apt update' (10/10) after sleeping 25 seconds
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Hit:5 http://security.ubuntu.com/ubuntu bionic-security InRelease
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.

Failed to fetch

Removing intermediate container 4ccbd94a5cf8
The command '/bin/sh -c python3 -u /tmp/wrapper_scripts/apt.py update-install-clean -q -y git mercurial python3-apt python3-catkin-pkg-modules python3-empy python3-rosdep python3-rosdistro-modules subversion wget' returned a non-zero code: 1
Build step 'Execute shell' marked build as failure
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 16251 killed;
[ssh-agent] Stopped.
```

`Step 19/26`에서 에러가 나는 것은 확인했는데, 에러 내용이 `apt-get update`입니다.

```
Step 19/26 : RUN python3 -u /tmp/wrapper_scripts/apt.py update-install-clean -q -y git mercurial python3-apt python3-catkin-pkg-modules python3-empy python3-rosdep python3-rosdistro-modules subversion wget
---> Running in 4ccbd94a5cf8
Invoking 'apt-get update'
Err:1 http://archive.ubuntu.com/ubuntu bionic InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:2 http://archive.ubuntu.com/ubuntu bionic-updates InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Err:3 http://archive.ubuntu.com/ubuntu bionic-backports InRelease
 Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
Hit:4 http://repositories.ros.org/ubuntu/testing bionic InRelease
Get:5 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Fetched 88.7 kB in 0s (235 kB/s)
Reading package lists...
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-updates/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Failed to fetch http://archive.ubuntu.com/ubuntu/dists/bionic-backports/InRelease  Something wicked happened resolving 'archive.ubuntu.com:80' (-5 - No address associated with hostname)
W: Some index files failed to download. They have been ignored, or old ones used instead.
```

도무지 모르겠다고 생각한 찰나, 패키지명 앞에 붙은 `Mdoc__`이라는 접두사가 신경쓰였습니다.

![Mdoc__ 실패 화면]()

![카테고리에 Mdev__가 있습니다.]()

빙고! 패키지 빌드에 실패한 이유는,.... 해당 패키지를 [wiki.ros.org](wiki.ros.org)에 등록하지 않아서였습니다. 옵션인줄로만 알았는데... 필수였네요.

![Mdev__ 성공 화면]()

`Mdev__`는 빌드에 성공한 모양입니다. [이전에 작성한 글](./day25_release_ROS_package_1.md)을 참고하여 위키에 등록합니다.

# 패키지를 indexer에 추가
이는 `bloom`을 통해 패키지를 Release하며 이미 완료되어 있는 사항입니다.

# 위키 페이지 생성하기
1.wiki.ros.org에 로그인합니다.

![로그인]()

![회원가입]()

2.위키 작성자로 등록되지 않았다면 아래 문구에서 `this GitHub ticket`을 눌러 다음 페이지에 자신의 위키 닉네임을 댓글로 남깁니다.

![this GitHub ticket]()

![GitHub Comment]()

Whitelist에 추가되기 전까지는 다음과 같이 위키 페이지를 수정할 수 없다는 표시가 생성됩니다. 저는 밤 12시에 신청하고 3시간 후에 승인되었습니다.

![not allowed]()

승인 이후에 `PackageTemplate`를 누르면 다음과 같은 템플릿이 표시됩니다.

![PackageTemplate]()

```
<<PackageHeader(@PAGE@)>>
<<TOC(4)>>




## AUTOGENERATED DON'T DELETE
## CategoryPackage

```

위키 작성 방법은 보통의 마크다운 문법과도 사뭇 다릅니다. 저는 헷갈리니까 다른 패키지의 내용을 복사-붙여넣기 하고 수정하도록 하겠습니다.

http://wiki.ros.org/dynamixel_sdk

오른편을 보면 `Edit (Text)`가 있습니다. 이를 누르면 작성된 위키 내용을 확인할 수 있습니다.

```
<<PackageHeader(dynamixel_sdk)>>
<<TOC(4)>>

''ROS Software Maintainer: [[http://wiki.ros.org/ROBOTIS|ROBOTIS]]''

== Overview ==
{{attachment:DYNAMIXEL_SDK_Logo.jpg}} 
{{attachment:DXL_SDK_image.jpg}}

This package wraps the '''ROBOTIS Dynamixel SDK''' to be available for ROS. The ROBOTIS Dynamixel SDK is a software development library that provides Dynamixel control functions for packet communication. The API is designed for Dynamixel actuators and Dynamixel-based platforms.

== ROBOTIS e-Manual ==
 * [[http://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/|ROBOTIS e-Manual for Dynamixel SDK]]

== ROS Wiki related to Dynamixel SDK ==
 * [[dynamixel_sdk]]
 * [[dynamixel_workbench]]
 * [[robotis_framework]]
 * [[turtlebot3]]
 * [[open_manipulator]]
 * [[robotis_op3]]
 * [[thormang3_mpc]]
 * [[manipulator_h]]
 * [[opencr]]

== References related to Dynamixel SDK ==
 * [[https://www.youtube.com/c/ROBOTISOpenSourceTeam|YouTube Channel of ROBOTIS OpenSourceTeam]]
 * [[https://community.robotsource.org/|Community for People Making Robots]]

== Video ==
 <<Youtube(F-sXbIAM0jc)>>
```

글 작성 중간중간에 `Preview` 버튼을 눌러 작성 중인 문서를 확인해볼 수 있습니다.

![code]()

![preview draft]()

글을 다 작성하였으면 저장한 뒤, `package.xml`에 `<url type="website">https://wiki.ros.org/turtle_teleop_multi_key</url>`와 같은 식으로 추가하여 줍니다. 해당 정보는 우리가 실패한 `jenkins` 작업에서 필요로 합니다. 이후 아래 명령어를 통해 변경 사항을 정리합니다.

```sh
catkin_prepare_release
vi CHANGELOG.rst
```

`package.xml`과 `CHANGELOG.rst`를 커밋한 뒤, `bloom-release`를 통해 다시 배포를 실시했습니다. 이제 `rosdistro`에서 `Pull Request`가 머지될 때까지 기다려야겠네요..