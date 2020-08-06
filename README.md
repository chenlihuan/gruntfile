# gruntfile
项目配置文件
module.exports = function(grunt) {

	var dateFormat = function(timeStamp) {
		var date = new Date(timeStamp);
		var Y = date.getFullYear(),
			M = (date.getMonth() + 1) < 10 ? '0' + (date.getMonth() + 1) : date.getMonth() + 1,
			D = date.getDate() < 10 ? '0' + date.getDate() : date.getDate(),
			h = date.getHours() < 10 ? '0' + date.getHours() : date.getHours(),
			m = date.getMinutes() < 10 ? '0' + date.getMinutes() : date.getMinutes(),
			s = date.getSeconds() < 10 ? '0' + date.getSeconds() : date.getSeconds();
		return Y + M + D + h + m + s;
	}
	grunt.initConfig({

		config: {
			src: 'source',
			dest: 'dist',
			tmp: '.tmp',
			date: dateFormat(new Date().getTime())
		},

		pkg: grunt.file.readJSON('package.json'),

		// Banner
		bannerCss: '/*!\n' +
			' * <%= pkg.name %> <%= grunt.template.today("isoDateTime") %> by <%= pkg.author %>\n' +
			' */',
		bannerJs: '/*!\n' +
			' * <%= pkg.name %> <%= grunt.template.today("isoDateTime") %> by <%= pkg.author %>\n' +
			' */',

		// Clean files and folders
		clean: {
			release: {
				src: ['<%= config.dest %>/**/*']
			}
		},

		// Grunt task to include files and replace variables
		includereplace: {
			options: {
				globals: {
					version: '<%= grunt.template.today("yyyymmddHHMMss") %>'
				}
			},
			html: {
				files: [{
					expand: true,
					cwd: '<%= config.src %>/pages/',
					src: '*.html',
					dest: '<%= config.dest %>/pages/'
				}]
			}
		},		
		
		sass: {
			options: {
				style: 'expanded',
				unixNewlines: true
			},
			dist: {
				files: [{
					expand: true,
					cwd: '<%= config.src %>/sass/',
					src: '*.scss',
					dest: '<%= config.dest %>/css/',
					ext: '.css'
				}]
			},

			modules: {
				files: [{
					expand: true,
					cwd: '<%= config.src %>',
					src: [
						'*/sass/*.scss',
						'!sass/**/*.*'
					],
					dest: '<%= config.dest %>/css',
					ext: '.css',
					flatten: true,
					filter: 'isFile'
				}]
			}
		},

		// Adds a simple banner to files
		usebanner: {
			distCss: {
				options: {
					position: 'top',
					banner: '<%= bannerCss %>'
				},
				files: {
					src: [
						'<%= config.dest %>/css/mui.min.css'
					]
				}
			},
			distJs: {
				options: {
					position: 'top',
					banner: '<%= bannerJs %>'
				},
				files: {
					src: [
						'<%= config.dest %>/js/plugin.js'
					]
				}
			}
		},

		// Copy files
		copy: {
			distJs: {
				files: [{
					expand: true,
					cwd: '<%= config.src %>',
					src: [
						'js/*.*',
						'js/zTree/**/*.*',
						'js/umeditor/**/*.*'
					],
					dest: '<%= config.dest %>'
				}]
			},
			distImg: {
				files: [{
					expand: true,
					cwd: '<%= config.src %>',
					src: [
						'images/**/*.*',						
						'fonts/**/*.*'
					],
					dest: '<%= config.dest %>'
				}]
			},
			commit: {
				files: [{
					expand: true,
					cwd: '<%= config.dest %>',
					src: '**/*.*',
					dest: './html/'
				}]
			}
		},

		// Sorting CSS properties in specific order
		csscomb: {
			dynamic_mappings: {
				expand: true,
				cwd: '<%= config.dest %>/css/',
				src: [
					'*.css',
					'!*.min.css'
				],
				dest: '<%= config.dest %>/css/'
			}			
		},

		// Minify images
		imagemin: {
			dynamic: {
				options: {
					optimizationLevel:16// PNG 图片优化水平
				},
				files: [{
					expand: true,
					cwd: "<%= config.dest %>/images/",
					src: ["**/*.{png,jpg,gif}"],
					dest: "<%= config.dest %>/images/"
				}]
			}
		},

		// Start a connect web server
		connect: {
			options: {
				port: 9520,
				hostname: "0.0.0.0",
				livereload: 35738
			},
			all: {
				options: {
					open: true,
					base: "<%= config.dest %>/"
				}
			}
		},

		// File revision based on content hashing
		filerev: {
			css: {
				src: '<%= config.dest %>/css/mui.min.css'
			},
			js: {
				src: [
					'<%= config.dest %>/js/**/{,*/}*.js',
					// --------------------------------
					'!<%= config.dest %>/js/jquery-1.9.1.min.js'					
				]
			}
		},

		// Run predefined tasks whenever watched file patterns are added, changed or deleted
		watch: {
			options: {
				spawn: false,
				livereload: '<%= connect.options.livereload %>' // this port must be same with the connect livereload port
			},
			gruntFile: {
				files: [
					'Gruntfile.js'
				],
				tasks: ['devHtml', 'devJs', 'devCss', 'devImg']
			},
			html: {
				files: [
					'<%= config.src %>/pages/**/*'
				],
				tasks: 'devHtml'
			},
			js: {
				files: [
					'<%= config.src %>/js/**/*',
					'Gruntfile.js'
				],
				tasks: 'devJs'
			},
			css: {
				files: [
					'<%= config.src %>/sass/**/*'
				],
				tasks: 'devCss'
			},
			img: {
				files: [
					'<%= config.src %>/images/**/*',
					'<%= config.src %>/fonts/**/*'
				],
				tasks: 'devImg'
			}
		},

		// Concatenate files
		concat: {
			mui: {
				src: [
				 '<%= config.src %>/js/....',		          
				],
				dest: '<%= config.dest %>/js/...'
			}
		},

		// Compress CSS files
		cssmin: {
			options: {
				keepSpecialComments: 0,
				report: 'min'
			},
			dist: {
				files: [{
					expand: true,
					cwd: '<%= config.dest %>/css/',
					src: [
						'*.css',
						'!*.min.css'
					],
					dest: '<%= config.dest %>/css/',
					ext: '.min.css'
				}]
			}			
		},

		postcss: {
			options: {
				map: {
					inline: false,
					annotation: '<%= config.dest %>/css/maps/'
				},
				processors: [
					require('autoprefixer')({
						browsers: [
							"Android >= 4",
							"Chrome >= 20",
							"Firefox >= 24",
							"iOS >= 6",
							"Safari >= 6"
						]
					})
					//,
					//require('cssnano')({
					//  discardComments: { removeAll: true }
					//})
				]
			},
			dist: {
				files: [{
					src: '<%= config.dest %>/css/style.css',
					dest: '<%= config.dest %>/css/style.css'
				}]
			}
		},

		// TODO
		// Spritesheet making utility
		sprite: {			
			front: {
				src: '<%= config.src %>/images/sprites/*.png',				
				retinaSrcFilter: '<%= config.src %>/images/sprites/*@2x.png',
				dest: '<%= config.src %>/images/sprites.png',				
				retinaDest: '<%= config.src %>/images/sprites-2x.png',
				destCss: '<%= config.src %>/sass/_sprites.scss',	
				cssTemplate: '<%= config.src %>/template/scss_retina.template.handlebars',				
				cssVarMap: function(sprite) {
					sprite.name = 'icon-' + sprite.name;
				}
			},
			huiyuan:{
				src: '<%= config.src %>/images/spriteshuiyuan/*.png',
				retinaSrcFilter: '<%= config.src %>/images/spriteshuiyuan/*@2x.png',
				dest: '<%= config.src %>/images/spriteshuiyuan.png',								
				retinaDest: '<%= config.src %>/images/spriteshuiyuan-2x.png',
				destCss: '<%= config.src %>/sass/_spriteshuiyuan.scss',	
				cssTemplate: '<%= config.src %>/template/scss_retina.template.handlebars',			
				cssVarMap: function(sprite) {
					sprite.name = 'iconh-' + sprite.name;
				}
			}
		}

	});

	// Time how long tasks take. Can help when optimizing build times
	require('time-grunt')(grunt);

	// Load grunt tasks automatically
	require('load-grunt-tasks')(grunt);

	// 配置任务
	grunt.registerTask('default', [
		'devHtml',
		'devJs',
		'devCss',
		'devImg',
		'connect:all',
		'watch'
	]);
	grunt.registerTask('devHtml', [
		'includereplace'
	]);
	grunt.registerTask('devJs', [
		'concat',
		'copy:distJs'
	]);
	grunt.registerTask('devCss', [
		'sass',
		'postcss',		
	]);
	grunt.registerTask('devImg', [
		'copy:distImg'
	]);
	grunt.registerTask('build', [
		'clean',
		'copy',
		'devHtml',
		'devJs',
		'devCss',
		'devImg',
		'postcss',
		'csscomb',
		'cssmin'
		//'usebanner:distCss',
		//'imagemin'
	]);
//	grunt.registerTask('commit', [
//		'copy:project'
//	])
};
