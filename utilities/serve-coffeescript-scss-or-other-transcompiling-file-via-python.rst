Serve CoffeeScript, SCSS, or other transcompiling file via Python
=================================================================

Usually it can be tiring to execute

::

    coffee -c


every time, even in watching mode. So you can disable the static file
serving by the code likes following.


::

    from subprocess import Popen, PIPE
    import shlex
    
    class NonZeroExitError(Exception):
        def __init__(self, command, returncode, output, error):
            self.command = command
            self.returncode = returncode
            self.output = output
            self.error = error
        def __str__(self):
            return '''\
            %s returned non-zero exit status %d with output
            %s
            and error
            %s''' % (self.command, self.returncode,
                     self.output or "[NO OUTOUT]",
                     self.error or "[NO ERROR]")
    
    def command_line_renderer_factory(command):
        '''command should be a command reads input from stdin
        and prints to stdout'''
        
        args = shlex.split(command)
        
        def renderer(script):
            '''Accepts a file object or path and return the
            rendered string'''
            
            if isinstance(script, file):
                pass
            elif isinstance(script, str):
                script = open(script)
            else:
                raise TypeError('script must be a file object of '
                                'or a string to the file')
            process = Popen(args, stdin=script,
                            stdout=PIPE, stderr=PIPE)
            
            returncode = process.wait()
            stdoutdata, stderrdata = process.communicate()
            
            if returncode != 0:
                raise NonZeroExitError(command, returncode,
                                       stdoutdata, stderrdata)
            
            return stdoutdata
        
        return renderer


and use them like:


::

    render_coffee = \
        command_line_renderer_factory('coffee -cs')
    render_raw_scss = \
        command_line_renderer_factory('sass -s --scss')


It will be a bit more complex to do this with LESS since the default
lessc does not read stdin. But you can write a compiler that reads
stdio like following, thanks Nikolay V. Nemshilov for the help of
STDIN at ( `http://st-on-it.blogspot.com/2011/05/how-to-read-user-
input-with-nodejs.html`_ ).


::

    #!/usr/bin/env node
    
    var less = require('less');
    
    process.stdin.resume();
    process.stdin.setEncoding('utf8');
    
    process.stdin.on('data', function (less_css) {
      less.render(less_css, function(e, css){
      	console.log(css)
      });
    });


But be sure never to use such a hack on production, you should compile
them and using the static ones.
.. _http://st-on-it.blogspot.com/2011/05/how-to-read-user-input-with-nodejs.html: http://st-on-it.blogspot.com/2011/05/how-to-read-user-input-with-nodejs.html

