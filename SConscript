# Copyright (c) 2012-2013, The Microsystems Design Labratory (MDL)
# Department of Computer Science and Engineering, The Pennsylvania State University
# All rights reserved.
# 
# This source code is part of NVMain - A cycle accurate timing, bit accurate
# energy simulator for both volatile (e.g., DRAM) and non-volatile memory
# (e.g., PCRAM). The source code is free and you can redistribute and/or
# modify it by providing that the following conditions are met:
# 
#  1) Redistributions of source code must retain the above copyright notice,
#     this list of conditions and the following disclaimer.
# 
#  2) Redistributions in binary form must reproduce the above copyright notice,
#     this list of conditions and the following disclaimer in the documentation
#     and/or other materials provided with the distribution.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# 
# Author list: 
#   Matt Poremba    ( Email: mrp5060 at psu dot edu 
#                     Website: http://www.cse.psu.edu/~poremba/ )

import os, sys
import subprocess

from os.path import basename


HG_COMMAND = 'hg'
if 'NVMAIN_HG' in os.environ:
    HG_COMMAND = os.environ['NVMAIN_HG']


Import('*')

# Assume that this is a gem5 extras build if this is set.
if 'TARGET_ISA' in env and env['TARGET_ISA'] == 'no':
    Return()

    env.Append(CPPPATH=Dir('.'))

if 'TARGET_ISA' in env and not 'NVMAIN_BUILD' in env:
    def NVMainSource(src):
        return Source(src)
    Export('NVMainSource')

    # Attempt to check the revision number of gem5
    print("Checking gem5 revision number...")

    # Assume some non-ancient version of gem5
    gem5_rv = 11654
    try:
        # Use 'hg log' to get qparent revision number information
        proc = subprocess.Popen([HG_COMMAND, 'log', '--template', '{rev}\n', '-r', 'qparent'], 
                                stdout=subprocess.PIPE, stderr = open(os.devnull, 'w'))
        proc.wait()

        # If the return code is not successful, assume no patches are applied
        if proc.returncode == 0:
            gem5_rv = int(proc.communicate()[0].rstrip())
            print(gem5_rv)
        elif proc.returncode == 255:
            # If a failure returned, we may not have any patches applied:
            # Try to get the revision number another way
            proc = subprocess.Popen([HG_COMMAND, 'identify', '--num'], 
                                    stdout=subprocess.PIPE, stderr = open(os.devnull, 'w'))
            proc.wait()

            if proc.returncode == 0:
                gem5_rv = int(proc.communicate()[0].rstrip())
                print(gem5_rv)
            else:
                print("Unknown. Assuming recent revision.")
        else:
            print("Unknown. Assuming recent revision.")
    except:
        gem5_rv = 11654
        pass

    gem5_rv_define = '-DNVM_GEM5_RV=' + str(gem5_rv)
    env.Append(CCFLAGS=gem5_rv_define)
    env['NVM_GEM5_RV'] = gem5_rv

# Common source files for any build
NVMainSource('NVM/nvmain.cpp')
NVMainSource('SimInterface/NullInterface/NullInterface.cpp')
NVMainSource('MemControl/MemoryControllerFactory.cpp')
NVMainSource('traceReader/TraceLine.cpp')

if 'NVMAIN_BUILD' in env:
    # NVMain build.
    NVMainSource('traceSim/traceMain.cpp')

    NVMainSource('traceReader/TraceReaderFactory.cpp')
    NVMainSource('traceReader/RubyTrace/RubyTraceReader.cpp')
    NVMainSource('traceReader/NVMainTrace/NVMainTraceReader.cpp')

elif 'TARGET_ISA' in env:
    # Assume that this is a gem5 extras build if this is set.
    NVMainSource('SimInterface/Gem5Interface/Gem5Interface.cpp')

    generated_dir = Dir('../protocol')

    def MakeIncludeAction(target, source, env):
        f = file(str(target[0]), 'w')
        for s in source:
            print >>f, '#include "%s"' % str(s.abspath)
        f.close()

    def MakeInclude(source):
        target = generated_dir.File(basename(source))
        include_action = MakeAction(MakeIncludeAction, Transform("MAKE INC", 1))
        env.Command(target, source, include_action)

    MakeInclude('SimInterface/Gem5Interface/Gem5Interface.h')
elif "NVMAINPATH" in os.environ:
    # Assume that this is a Zsim build if this is set.
    # Nothing to be done here for now.
    pass
else:
    print("ERROR: What kind of build is this?")
    sys.exit(1)

