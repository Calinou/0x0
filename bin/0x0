#!/usr/bin/env bash
#
# Copyright © 2018-2019 Hugo Locurcio and contributors
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

set -euo pipefail
IFS=$'\n\t'

API_ENDPOINT=https://0x0.st/
EXPIRES=0
INPUT_ARG=""
MIMETYPE=""
VERBOSE=false

if [[ -t 1 ]]; then
  # stdout is a TTY, enable colors
  STYLE_COMMAND="\033[0;97m"
  STYLE_ARG="\033[0;96m"
  STYLE_ERROR="\033[91m"
  STYLE_BOLD="\033[1m"
  STYLE_NORMAL="\033[22m"
  STYLE_RESET="\033[0m"
else
  # stdout is not a TTY, disable colors
  STYLE_COMMAND=""
  STYLE_ARG=""
  STYLE_ERROR=""
  STYLE_BOLD=""
  STYLE_NORMAL=""
  STYLE_RESET=""
fi

# Prints an error message to stderr
print_error() {
  echo -e "${STYLE_ERROR}${STYLE_BOLD}Error:${STYLE_NORMAL} $1${STYLE_RESET}" >&2
  exit 1
}

print_help() {
  base="${0##*/}"
  echo -e "${STYLE_BOLD}0x0, a wrapper script for ${API_ENDPOINT}${STYLE_RESET}\n"
  echo -e "Usage: ${base} [options] <file|url>\n"
  echo -e "Options:"
  echo -e "  -e | --expires ${STYLE_ARG}<number>${STYLE_RESET}  Set an expiration on the file"
  echo -e "                           'number' is the number of hours from now or"
  echo -e "                           a timestamp in epoch milliseconds."
  echo -e "  -m | --mimetype ${STYLE_ARG}<type>${STYLE_RESET}   Force a mime-type to the uploaded content"
  echo -e "                           example: -m text/plain"
  echo -e "  -v | --verbose           Also return relevant headers from response"
  echo -e "                           Outputs x-expires and x-token headers if present"
  echo -e "Examples:"
  echo -e "  Upload a file:"
  echo -e "      ${STYLE_COMMAND}${base} ${STYLE_ARG}<file>${STYLE_RESET}\n"
  echo -e "  Upload from an URL (the file won't be fetched locally):"
  echo -e "      ${STYLE_COMMAND}${base} ${STYLE_ARG}<url>${STYLE_RESET}\n"
  echo -e "  Upload from standard input:"
  echo -e "      ${STYLE_COMMAND}${base} ${STYLE_ARG}-${STYLE_RESET}\n"
  echo -e "The uploaded file's URL is printed to standard output when the upload is completed."
}

# Process options and arguments
while [[ $# -gt 0 ]]; do
  opt="$1"
  shift;
  case "$opt" in
    "-e"|"--expires")
      if [[ $# -eq 0 ]] || [[ "$1" =~ ^-{1,2}.* ]]; then
        print_error "Missing or invalid argument for expires option"
      fi
      EXPIRES="$1"
      is_number='^[0-9]+$'
      if ! [[ $EXPIRES =~ $is_number ]] ; then
        print_error "Expires value must be a number"
      fi
      shift
      ;;
    "-h"|"--help")
      print_help
      exit 0
      ;;
    "-m"|"--mimetype")
      if [[ $# -eq 0 ]] || [[ "$1" =~ ^-{1,2}.* ]]; then
        print_error "Missing or invalid argument for mimetype option"
      fi
      MIMETYPE="$1"
      ;;
    "-v"|"--verbose")
      VERBOSE=true
      ;;
    *)
      INPUT_ARG="$opt"
      ;;
  esac
done

# Did we get an input argument?
if [[ "$INPUT_ARG" == "" ]]; then
  # No input argument
  print_help
  exit 1
fi

# Construct an array of opts for the curl command
curl_opts=( "$API_ENDPOINT" )

# Add support for verbose logic to curl command
if [[ $VERBOSE == true ]]; then
  tmp_file=$(mktemp)
  curl_opts=( --no-progress-meter --dump-header "$tmp_file" "${curl_opts[@]}")
fi

# Add expires to curl opts
if [[ $EXPIRES -gt 0 ]]; then
  curl_opts=( -F expires="$EXPIRES" "${curl_opts[@]}" )
fi

# Make appropriate curl opts from input
if [[ -f "$INPUT_ARG" || "$INPUT_ARG" = '-' ]]; then
  # Upload from file or stdin
  if [[ $MIMETYPE != "" ]]; then
    INPUT_ARG="$INPUT_ARG;type=$MIMETYPE"
  fi
  curl_opts=( -F file=@"$INPUT_ARG" "${curl_opts[@]}" )

elif [[ "$INPUT_ARG" =~ ^https?://.* ]]; then
  # Upload from URL
  curl_opts=( -F url="$INPUT_ARG" "${curl_opts[@]}" )

elif [[ -d "$INPUT_ARG" ]]; then
  print_error "\"$INPUT_ARG\" is a directory."

else
  print_error "\"$INPUT_ARG\": no such file."
fi

curl "${curl_opts[@]}"

if [[ $VERBOSE == true ]]; then
  echo -n -e "${STYLE_ARG}"
  < "$tmp_file" grep -e x-expires -e x-token | cat
  echo -n -e "${STYLE_RESET}"

  rm -f "$tmp_file"
fi
