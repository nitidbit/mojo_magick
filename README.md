MojoMagick
==========

MojoMagick is a "dog simple, do very little" image library. It is basically a couple of stateless
module methods that make it somewhat more convenient than calling ImageScience by hand.

The main reason to use MojoMagick is that you should consolidate your ImageScience calls into
one place, so why not do it here? If you improve on this tool, send me the patch.

This tool came about because I wanted a fast, simple, lightweight, nothing-goes-wrong-with-it-
because-it's-too-simple-to-break image tool.

Examples
========

## Image Resizing

### Obtain the size of an image (assuming image is "120wx222h")

    dimensions = MojoMagick::get_image_size(test_image)
    # ==> dimensions now holds a hash: {:height => 120, :width => 222}

### Resize an image so that it fits within a 100w x 200h bounding box
(Note: this will scale an image either up or down to fit these dimensions which may not be what you want.)
In this example, we overwrite our image, but if you pass in a different file for the second file name, a new file will be created with the resized dimensions

    MojoMagick::resize('/img/test.jpg', '/img/test.jpg', {:width=>100, :height=>200})

### Resize an image so that it fills a 100 x 100 bounding box

After this transformation, your image's short side will be 100px. This will preserve the aspect ratio.

    MojoMagick::resize('/img/infile.jpg', '/img/outfile.jpg', {:width=>100, :height=>100, :fill => true})

### Resize an image so that it fills and is cropped to a 100 x 100 bounding box

After this transformation, your image will be 100x100.  But it does not distort the images.  It crops out of the Center.

    MojoMagick::resize('/img/infile.jpg', '/img/outfile.jpg', {:width=>100, :height=>100, :fill => true, :crop => true})

### Code sample of how to shrink all jpg's in a folder

    require 'mojo_magick'
    
    image_folder = '/tmp/img'
    Dir::glob(File::join(image_folder, '*.jpg')).each do |image|
      begin
        # shrink all the images *in place* to no bigger than 60pix x 60pix
        MojoMagick::shrink(image, image, {:width => 60, :height => 60})
        puts "Shrunk: #{image}"
      rescue MojoMagick::MojoFailed => e
        STDERR.puts "Unable to shrink image '#{image}' - probably an invalid image\n#{e.message}"
      rescue MojoMagick::MojoMagickException => e
        STDERR.puts "Unknown exception on image '#{image}'\n#{e.message}"
      end
    end

## Setting Memory/Resource limits for ImageMagick
Be sure you're upgraded to the current release of ImageMagick.

set limits on disk, area, map and ram usage
obtain/print a hash of default limits:

    puts MojoMagick::get_default_limits.inspect

current\_limits shows same values:

    puts MojoMagick::get_current_limits.inspect

    MojoMagick::set_limits(:area => '32mb', :disk => '0', :memory => '64mb', :map => '32mb')
    puts MojoMagick::get_current_limits.inspect

As of ImageMagick 6.6, you also have the ability to set `:threads` and `:time`.  Read [ImageMagick docs on limits](http://www.imagemagick.org/script/command-line-options.php#limit) for more info.

## For more complex operations (thanks to Elliot Nelson for adding this code to the system)

Two command-line builders, #convert and #mogrify, have been added to simplify
complex commands.

### using #convert

    MojoMagick::convert('source.jpg', 'dest.jpg') do |c|
      c.crop '250x250+0+0'
      c.repage!
      c.strip
      c.set 'comment', 'my favorite file'
    end
    
    # Equivalent to:
    MojoMagick::raw_command('convert', 'source.jpg -crop 250x250+0+0 +repage -strip -set comment "my favorite file" dest.jpg')


### using #mogrify

    MojoMagick::mogrify('image.jpg') {|i| i.shave '10x10'}

    # Equivalent to:

    MojoMagick::raw_command('mogrify', '-shave 10x10 image.jpg')

    # Example showing some additional options:
    # assuming binary data that is rgb, 8bit depth and 10x20 pixels, :format => :rgb, :depth => 8, :size => '10x20'OAu
    
    MojoMagick::convert do |c|
      c.file 'source.jpg'
      c.blob my_binary_data, :format => :rgb, :depth => 8, :size => '10x20'
      c.append
      c.crop '256x256+0+0'
      c.repage!
      c.file 'output.jpg'
    end
    
    # Use .file to specify file names, .blob to create and include a tempfile. The
    # bang (!) can be appended to command names to use the '+' versions
    # instead of '-' versions.

### Create a brand new image from data

    binary_data = '1111222233334444'
    MojoMagick::convert do |c|
      c.rgba8 binary_data, :format => :rgba, :depth => 8, :size => '2x2'
      c.file 'output.jpg'
    end

Availablility
=============

 * [SVN Repo access](http://trac.misuse.org/science/wiki/MojoMagick)
 * Concact the author or discuss MojoMagick : [misuse.org](http://www.misuse.org/science/2008/01/30/mojomagick-ruby-image-library-for-imagemagick/)


#### Updates by Jon Rogers Aug 2011 (http://github.com/bunnymatic)
* added gemspec for building gem with bundler
* updated tests for ImageMagick 6.6
* added ability to do fill + crop resizing
* bumped version to 0.3.0
* [new github repo](https://github.com/bunnymatic/mojo_magick)


References
==========

* [ImageMagick](http://www.imagemagick.org/)

Copyright (c) 2008 Steve Midgley, released under the MIT license
 Credit to Elliot Nelson for significant code contributions. Thanks Elliot!
