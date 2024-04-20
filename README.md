# Reproduction of `git diff --exit-code` bug

This repository sets up a simple scenario to reproduce a bug in `git diff --exit-code`.
The bug is that `git diff --exit-code` returns 1 even when there are no changes in shown in the output.
It seems that the status code is set based on the list of changed files rather than the actual changes,
that can be modified by using diff filters via the gitattributes mechanism.

## Steps to reproduce

1. Clone this repository
1. Configure the diff filter:

   ```shell
   cat <<EOF > .git/config
   [diff "equalize"]
       textconv = ./equalizer
   EOF
   ```

1. Run `git diff --exit-code`:

   1. Correct behavior: returning `0` when there are no changes

      ```console
      $ git diff --exit-code HEAD~1 -- equalizer

      $ echo $?
      0
      ```

   1. Correct behavior: returning `1` when there are changes in the output

      ```console
      $ git diff --exit-code HEAD~1 -- good.txt
      diff --git a/good.txt b/good.txt
      index b2a7546..f761ec1 100644
      --- a/good.txt
      +++ b/good.txt
      @@ -1 +1 @@
      -ccc
      +bbb

      $ echo $?
      1
      ```

   1. Incorrect behavior: returning `1` when there are no changes in the output

      ```console
      $ git diff --exit-code HEAD~1 -- bad.txt

      $ echo $?
      1
      ```
