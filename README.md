wp-zip-generator
================

Zip File Generator Class for WordPress

## What It Does ##

The WP Zip Generator automates a bunch of tasks related to creating a zip file that a user can download. 
The "source" directory is recursively iterated through and each file is processed. 
You can specify which files and directories to exclude from the final zip.
When each file is processed, a regex replace is performed on the file contents using variables that you can specify.

## Example Usage ##

```
//setup some variables for replacement
$variables = array(
	'{name}'               => 'Double Rainbow',
	'{slug}'               => 'double-rainbow'
	'{author}'             => 'Brad Vincent',
	'{base-color}'         => '#222',
	'{highlight-color}'    => '#d00',
	'{notification-color}' => '#369',
	'{action-color}'       => '#080'
);

//create the generator
$zip_generator = new WP_Zip_Generator(array(
	'name'                 => 'admin-color-scheme-generator',
	'process_extensions'   => array('php', 'css', 'js', 'txt', 'md'),	
	'source_directory'     => dirname( __FILE__ ) . '/source/',
	'zip_root_directory'   => "color-scheme",
	'download_filename'    => "double-rainbow.zip",
	'filename_filter'      => array($this, 'process_zip_filename'),
	'file_contents_filter' => array($this, 'process_zip_file_contents'),
	'post_process_action'  => array($this, 'process_zip'),
	'variables'            => $variables
));

//generate the zip file
$zip_generator->generate();

//download it to the client
$zip_generator->send_download_headers();

die();
```

## Arguments ##

The default arguments that are used in the generator are:

```
$defaults = array(
	'name'                 => '',
	'source_directory'     => '',
	'process_extensions'   => array('php', 'css', 'js', 'txt', 'md'),
	'zip_root_directory'   => '',
	'zip_temp_directory'   => plugin_dir_path( __FILE__ ),
	'download_filename'    => '',
	'exclude_directories'  => array('.git', '.svn', '.', '..'),
	'exclude_files'        => array('.git', '.svn', '.DS_Store', '.gitignore', '.', '..'),
	'filename_filter'      => null,
	'file_contents_filter' => null,
	'post_process_action'  => null,
	'variables'            => array()
);
```

## Hooks ##

You can easily hook into the zip generator to alter the way it works.

### filename_filter ###
Use this filter to change the file names in the final zip. Example:

```
function process_zip_filename($filename) {
	if ( 'plugin.php' === $filename ) {
		return "double-rainbox.php";
	}
	return $filename;
}
```

### file_contents_filter ###
Use this filter to change the contents of a file in the final zip. Example:

```
function process_zip_file_contents($contents, $filename) {
	if ( 'awesome.txt' === $filename ) {
		return 'Awesome content goes here!';
	}

	return $contents;
}
```

### post_process_action ###
This action fires after all files have been process from the source directory. Use this action to alter the final zip in any way. Example:

```
function process_zip($zip, $options) {
	//add a license file to the zip
	
	$license_contents = 'GPL2!;

	$zip->addFromString( 'license.txt', $license_contents );
}
```