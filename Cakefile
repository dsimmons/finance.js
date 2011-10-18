fs = require 'fs'
util = require 'util'
require 'colors'

{exec} = require 'child_process'
{basename, join} = require 'path'

srcCoffeeDir = 'src'
targetJsDir = 'lib'

outputFileName = 'node-finance'

coffeeOpts = "--bare --output #{targetJsDir} --compile #{srcCoffeeDir}"
coffeeOptsBare = "--bare"
coffeeOptWatch = "--watch"

appFiles = []

# Compile src/*.coffee to lib/*.js 

task 'coffeeFiles', 'How many coffee scripts do we have?', ->
	traverseFileSystem = (currentPath) ->
		files = fs.readdirSync currentPath
		for file in files
			do (file) ->
				currentFile = currentPath + '/' + file
				stats = fs.statSync(currentFile)
				if stats.isFile() and currentFile.indexOf('.coffee') > 1 and appFiles.join('=').indexOf("#{currentFile}=") < 0
					appFiles.push currentFile
				else if stats.isDirectory()
					traverseFileSystem currentFile
	
	traverseFileSystem "#{srcCoffeeDir}"
	util.log "#{appFiles.length} coffee files found.".grey
	return appFiles

task 'build', 'Build single application file from source files', ->
	invoke 'coffeeFiles'
	appContents = new Array 
	remaining = appFiles.length
	for file, index in appFiles then do (file, index) ->
		fs.readFile file, 'utf8', (err, fileContents) ->
			throw err if err
			appContents[index] = fileContents
			process() if --remaining is 0
	process = ->
		fs.writeFile outputFileName+".coffee", appContents.join('\n\n'), 'utf8', (err) ->
			throw err if err
			exec "coffee --bare --compile #{outputFileName+'.coffee'}", (err, stdout, stderr) ->
				if err
					util.log 'Error compiling coffee file.'.bold.red
				else
					fs.unlink outputFileName+".coffee", (err) ->
						if err
							util.log "Could't delete the #{outputFileName}.coffee file.".bold.yellow
					util.log 'Done building coffee file.'.green

task 'watch', 'Watch source files and build changes', ->
	invoke 'build'
	util.log "Watching for changes in #{srcCoffeeDir}".grey
	for file in appFiles then do (file) ->
		fs.watchFile file, (curr, prev) ->
			if +curr.mtime isnt +prev.mtime
				util.log "Saw change in #{file}".grey
				invoke 'build'