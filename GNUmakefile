CC=cl.exe
RC=rc.exe
LIB=lib.exe
MSBUILD=msbuild.exe
CMAKE=cmake.exe
RFLAGS=/nologo
CFLAGS=/nologo /Zi /Od /MD /FS
LFLAGS=/nologo /machine:x86
VFLAGS=-no_logo
MFLAGS=/p:Configuration=Release /nologo /m /v:q
CXXFLAGS=/nologo /Zi /Od /EHsc /MD /FS
LDLIBS=user32 ole32 edit advapi32 peparse shlwapi imm32 shell32
LDFLAGS=/MD
LINKFLAGS=/ignore:4099
VSDEVCMD=cmd.exe /c vsdevcmd.bat

# Commands for arch specific compiler.
CC64=$(VSDEVCMD) $(VFLAGS) -arch=amd64 "&" cl
CC32=$(VSDEVCMD) $(VFLAGS) -arch=x86 "&" cl

.PHONY: clean distclean

all: ctftool.exe payload32.dll payload64.dll

release: ctftool.zip ctftool-src.zip

%.res: %.rc
	$(RC) $(RFLAGS) $<

%.obj: %.cc
	$(CC) $(CXXFLAGS) /c /Fo:$@ $<

%.obj: %.c
	$(CC) $(CFLAGS) /c /Fo:$@ $<

%.exe: %.obj
	$(CC) $(CFLAGS) $(LDFLAGS) /Fe:$@ $^ /link $(LINKFLAGS) $(LDLIBS:=.lib)

%.dll: %.obj
	$(CC) $(CFLAGS) $(LDFLAGS) /LD /Fe:$@ $^ /link $(LINKFLAGS)

%64.obj: %.c
	$(CC) $(CFLAGS) /c /Fd:$(@:.obj=.pdb) /Fo:$@ $<

%32.obj: %.c
	$(CC) $(CFLAGS) /c /Fd:$(@:.obj=.pdb) /Fo:$@ $<

%64.dll: CC=$(CC64)
%64.dll: %64.obj
	$(CC) $(CFLAGS) $(LDFLAGS) /LD /Fd:$(@:.dll=.pdb) /Fe:$@ $^ /link $(LINKFLAGS)

%32.dll: CC=$(CC32)
%32.dll: %32.obj
	$(CC) $(CFLAGS) $(LDFLAGS) /LD /Fd:$(@:.dll=.pdb) /Fe:$@ $^ /link $(LINKFLAGS)

peparse.lib:
	$(CMAKE) -S pe-parse -B build-$@
	$(MSBUILD) $(MFLAGS) build-$@/pe-parse.sln
	cp build-$@/pe-parser-library/Release/pe-parser-library.lib $@

edit.lib:
	$(CMAKE) -S wineditline -B build-$@
	$(MSBUILD) $(MFLAGS) build-$@/WinEditLine.sln
	cp build-$@/src/Release/edit_a.lib $@

ctftool.exe: command.obj ctftool.obj winmsg.obj marshal.obj     \
             util.obj module.obj version.res peproc.obj         \
             messages.obj winutil.obj                           \
                | edit.lib peparse.lib

clean:
	rm -rf *.exp *.exe *.obj *.pdb *.ilk *.xml build-*.* *.res *.ipdb *.iobj *.dll *.tmp

# These are slow to rebuild and I dont change them often.
distclean: clean
	rm -f edit.lib peparse.lib
	rm -f ctftool.zip ctftool-src.zip

ctftool.zip: README.md ctftool.exe payload32.dll payload64.dll scripts docs
	(cd .. && zip -r ctftool/$@ $(patsubst %,ctftool/%,$^))

ctftool-src.zip:
	(cd .. && zip -x@ctftool/.zipignore -r ctftool/$@ ctftool)
