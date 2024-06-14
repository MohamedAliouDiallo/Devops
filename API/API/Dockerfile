# TODO: Choose a java JRE
FROM openjdk:20

# TODO:  Add the compiled java (aka bytecode, aka .class)
COPY Main.class /usr/src/app/

# TODO: Run the Java with: “java Main” command.
# CMD ["java", "/usr/src/app/Main"]
CMD ["java", "-cp", "/usr/src/app/", "Main"]