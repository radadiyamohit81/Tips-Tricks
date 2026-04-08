# From the project root folder:

java compile                    # compiles ALL .java files → out/
java compile ./Main.java        # compiles ONE specific file → out/
java run                        # runs Main class from out/

# Normal java still works (pass-through):
java --version                  # works as usual
java -jar something.jar         # works as usual



java() {
  if [[ "$1" == "compile" ]]; then    # intercept "compile"
    mkdir -p out                       # create out/ if missing
    if [[ -n "$2" ]]; then             # if a file was passed as arg
      javac -d out "$2"                # compile just that file
    else
      find . -name "*.java" | xargs javac -d out   # compile all
    fi
  elif [[ "$1" == "run" ]]; then       # intercept "run"
    command java -cp out Main          # run Main from out/
  else
    command java "$@"                  # everything else → real java
  fi
}
