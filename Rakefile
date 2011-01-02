# -*- ruby -*-
# Rakefile: build ruby libvirt bindings
#
# Copyright (C) 2007,2010 Red Hat, Inc.
#
# Distributed under the GNU Lesser General Public License v2.1 or later.
# See COPYING for details
#
# David Lutterkort <dlutter@redhat.com>

# Rakefile for ruby-rpm -*- ruby -*-
require 'rake/clean'
require 'rake/rdoctask'
require 'rake/testtask'
require 'rake/gempackagetask'

PKG_NAME='ruby-libvirt'
PKG_VERSION='0.3.0'

EXT_CONF='ext/libvirt/extconf.rb'
MAKEFILE="ext/libvirt/Makefile"
LIBVIRT_MODULE="ext/libvirt/_libvirt.so"
SPEC_FILE="ruby-libvirt.spec"
LIBVIRT_SRC=Dir.glob("ext/libvirt/*.c")
LIBVIRT_SRC << MAKEFILE

#
# Additional files for clean/clobber
#

CLEAN.include [ "ext/**/*.o", LIBVIRT_MODULE,
                "ext/**/depend" ]

CLOBBER.include [ "config.save", "ext/**/mkmf.log", "ext/**/extconf.h",
                  MAKEFILE ]

#
# Build locally
#
file MAKEFILE => EXT_CONF do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "ruby #{File::basename(EXT_CONF)}"
             $stderr.puts "Failed to run extconf"
             break
         end
    end
end
file LIBVIRT_MODULE => LIBVIRT_SRC do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
         unless sh "make"
             $stderr.puts "make failed"
             break
         end
     end
end
desc "Build the native library"
task :build => LIBVIRT_MODULE

#
# Test task
#

Rake::TestTask.new(:test) do |t|
    t.test_files = [ 'tests/test_conn.rb', 'tests/test_domain.rb', 'tests/test_interface.rb', 'tests/test_network.rb', 'tests/test_nodedevice.rb', 'tests/test_nwfilter.rb', 'tests/test_open.rb', 'tests/test_secret.rb', 'tests/test_storage.rb' ]
    t.libs = [ 'lib', 'ext/libvirt' ]
end
task :test => :build

Rake::RDocTask.new do |rd|
    rd.main = "README.rdoc"
    rd.rdoc_dir = "doc/site/api"
    rd.rdoc_files.include("README.rdoc", "lib/libvirt.rb", ["ext/libvirt/_libvirt.c", "ext/libvirt/connect.c", "ext/libvirt/domain.c", "ext/libvirt/interface.c", "ext/libvirt/network.c", "ext/libvirt/nodedevice.c", "ext/libvirt/nwfilter.c", "ext/libvirt/secret.c", "ext/libvirt/storage.c"])
end

Rake::RDocTask.new(:ri) do |rd|
    rd.main = "README.rdoc"
    rd.rdoc_dir = "doc/ri"
    rd.options << "--ri-system"
    rd.rdoc_files.include("README.rdoc", "lib/libvirt.rb", ["ext/libvirt/_libvirt.c", "ext/libvirt/connect.c", "ext/libvirt/domain.c", "ext/libvirt/interface.c", "ext/libvirt/network.c", "ext/libvirt/nodedevice.c", "ext/libvirt/nwfilter.c", "ext/libvirt/secret.c", "ext/libvirt/storage.c"])
end

#
# Splint task
#

task :splint => [ MAKEFILE ] do |t|
    Dir::chdir(File::dirname(EXT_CONF)) do
        unless sh "splint -I /usr/lib/ruby/1.8/i386-linux *.c"
            $stderr.puts "Failed to run splint"
            break
        end
    end
end

#
# Package tasks
#

PKG_FILES = FileList[
  "Rakefile", "COPYING", "README", "NEWS", "README.rdoc",
  "lib/**/*.rb",
  "ext/**/*.[ch]", "ext/**/MANIFEST", "ext/**/extconf.rb",
  "tests/**/*",
  "spec/**/*"
]

DIST_FILES = FileList[
  "pkg/*.src.rpm",  "pkg/*.gem",  "pkg/*.zip", "pkg/*.tgz"
]

SPEC = Gem::Specification.new do |s|
    s.name = PKG_NAME
    s.version = PKG_VERSION
    s.email = "libvir-list@redhat.com"
    s.homepage = "http://libvirt.org/ruby/"
    s.summary = "Ruby bindings for LIBVIRT"
    s.files = PKG_FILES
    s.required_ruby_version = '>= 1.8.1'
    s.extensions = "ext/libvirt/extconf.rb"
    s.author = "David Lutterkort, Chris Lalancette"
    s.rubyforge_project = "None"
    s.description = "Ruby bindings for libvirt."
end

Rake::GemPackageTask.new(SPEC) do |pkg|
    pkg.need_tar = true
    pkg.need_zip = true
end

desc "Build (S)RPM for #{PKG_NAME}"
task :rpm => [ :package ] do |t|
    system("sed -e 's/@VERSION@/#{PKG_VERSION}/' #{SPEC_FILE} > pkg/#{SPEC_FILE}")
    Dir::chdir("pkg") do |dir|
        dir = File::expand_path(".")
        system("rpmbuild --define '_topdir #{dir}' --define '_sourcedir #{dir}' --define '_srcrpmdir #{dir}' --define '_rpmdir #{dir}' --define '_builddir #{dir}' -ba #{SPEC_FILE} > rpmbuild.log 2>&1")
        if $? != 0
            raise "rpmbuild failed"
        end
    end
end
