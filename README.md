# script compression statics (PYTHON)

## script utilisé pour la compression des assets statics
#### Utilisé en conjonction d'un htaccess (voir plus bas)

````py
# coding: utf-8

# ------------------------------------------------------
# ----- recompresse recursivement png
# -----
# ------------------------------------------------------

from subprocess import call
import os
import sys
import re
import gzip
import json

compjs      = True
minjs       = True # minifie le JS
compcss     = True
mincss      = True # minifie le css
comppng     = True
compwebp    = True # compress png jpeg --> .png.webp jpeg.webp
compsvg     = True # minification des svg impossible
compjson    = True # minification json impossible
compgz      = True
compbro     = True # compression brotli
compfont    = True # compression des fontes
compjpg2wp  = True # compression jpeg en webp

ignore =  ['.eslintrc.js','#old','_fake','_inc','_json','_services','_sharing','_teamfifa','_trophee','export_internautes','import_joueurs']
#ignore =  ['_commonimg','_css','_img','_img_fr','_js','_sounds','_src','_tracking','others']

optionpng = ' --force --ext .png --quality=60-90 '
optionwebp = ' -q 50 -m 6 -alpha_q 50 -pass 6 -quiet '

optioncss = ' --type css '
optionjs = ' --type js '
nooption = ' '

sizepngo = 0 # size PNG originelle
sizepngc = 0 # size PNG recompresse

sizecsso = 0 # size CSS originelle
sizecssc = 0 # size CSS recompresse

sizejso = 0 # size JS originelle
sizejsc = 0 # size JS recompresse

regcss = re.compile('^(.*\.css)$')
regjs = re.compile('^(.*\.js)$')
reggz = re.compile('^(.*\.gz)$')
regpng = re.compile('^(.*\.png)$')
regsvg = re.compile('^(.*\.svg)$')
regbr = re.compile('^(.*\.br)$')
regjson = re.compile('^(.*\.json)$')
regwebp = re.compile('^(.*\.webp)$')
regjpeg = re.compile('^(.*\.jpg|.*\.jpeg)$')

# fonts
regfont = re.compile('^(.*\.otf|.*\.eot|.*\.ttf|.*\.woff|.*\.woff2)$')

regmincss = re.compile('^(.*\.min\.css)$') # supprime les fichier min.css précédents
regminjs = re.compile('^(.*\.min\.js)$') # supprime les fichier min.js précédents
regminsvg = re.compile('^(.*\.min\.svg)$') # supprime les fichier min.svg précédents
#reggz = re.compile('^(.*\.gz)$')
reggz = re.compile('^(.*\.min\.css.gz|.*\.min\.js.gz|.*\.svg.gz)$') # supprime les fichier gz précédents

compressed = []


weigths = {'css':{'before':0,'after':0,'gz':0,'br':0},\
           'js':{'before':0,'after':0,'gz':0,'br':0},\
           'svg':{'before':0,'after':0,'gz':0,'br':0},\
           'png':{'before':0,'after':0},\
           'json':{'before':0,'after':0,'gz':0,'br':0},\
           'otf':{'before':0,'after':0,'gz':0,'br':0},\
           'eot':{'before':0,'after':0,'gz':0,'br':0},\
           'ttf':{'before':0,'after':0,'gz':0,'br':0},\
           'woff':{'before':0,'after':0,'gz':0,'br':0},\
		   'woff2':{'before':0,'after':0,'gz':0,'br':0},\
           'webp':{'before':0,'after':0},\
           'jpegwebp':{'before':0,'after':0}}



# load list compressed files (no recompress)
fc = open('./compressed.dct','r');
temp = fc.read()
fc.close()
if temp != '':
    compressed = eval(temp)
    temp = ''


def addsuffx(suffx, opt, cfile, paths, pathfile):

    # mv en min.xx -----------------------------------------------------
    # suppr .min.xx existant

    filename = os.path.splitext(cfile)[0]
    extension = os.path.splitext(cfile)[1]
    # copie fichier en .min.css
    key = suffx[5:]

	# compresser UNIQUEMENT JS & CSS ------------------------------------
    # if key != 'svg' and key != 'json' and key != 'otf' and key != 'eot' and key != 'ttf' and key != 'woff':
    if (key == 'css' and mincss == True) or (key == 'js' and minjs == True) :
        filemin = paths + filename + suffx
        os.system('cp ' + pathfile + ' ' + filemin)
        # compression yui
        #sizecsso += os.path.getsize(filemin)
        option = opt + filemin + ' -o ' + filemin
        os.system('yuicompressor'+option)
        compressed.append(filemin)
    else:
        filemin = pathfile


    if key in weigths:
        weigths[key]['after'] += os.path.getsize(filemin)
	# ------------------------------------------------------------------



	# compression gz  -------------------------------------------------
    if compgz == True:
        f_in = open(filemin, 'rb')
        f_out = gzip.open(paths + filename + '.' + key + '.gz', 'wb')
        f_out.writelines(f_in)
        f_out.close()
        f_in.close()
        compressed.append(paths + filename + '.' + key + '.gz')
        if key in weigths:
            weigths[key]['gz'] += os.path.getsize(paths + filename + '.' +key+'.gz')
	# ------------------------------------------------------------------


	# compression brotli (https)  --------------------------------------
    if compbro == True:
        option = ' --force -q 11 -o ' +  paths + filename + '.' + key + '.br ' + filemin
        os.system('brotli'+option)
        compressed.append(paths + filename + '.' +key+'.br')

        if key in weigths:
            weigths[key]['br'] += os.path.getsize(paths + filename + '.' +key+'.br')
	# ------------------------------------------------------------------







# suppression des min.xx et min.gz et webp---------------------------

for root, dirs, files in os.walk("."):
    path = root.split(os.sep)
    for cfile in files:
        paths = os.path.abspath(root)+'/'
        pathfile = paths + cfile

        if reggz.search(cfile) is not None and pathfile not in compressed:
            os.system('rm ' + pathfile)

        if regbr.search(cfile) is not None and pathfile not in compressed:
            os.system('rm ' + pathfile)

        if mincss == True and regmincss.search(cfile) is not None and pathfile not in compressed:
            os.system('rm ' + pathfile)

        if minjs == True and regminjs.search(cfile) is not None and pathfile not in compressed:
            os.system('rm ' + pathfile)

        if regminsvg.search(cfile) is not None and pathfile not in compressed:
            os.system('rm ' + pathfile)

        if regwebp.search(cfile) is not None and pathfile not in compressed:
            os.system('rm ' + pathfile)







# compression --------------------------------------------------

for root, dirs, files in os.walk("."):
    path = root.split(os.sep)
    for cfile in files:
        paths = os.path.abspath(root)+'/'
        ignorerep = False
        if len(ignore) > 0:
            for i in ignore:
                if i in paths:
                    ignorerep = True
            if ignorerep == True: continue

        pathfile = paths + cfile


        if compcss == True:
            if regcss.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est une CSS !'
                weigths['css']['before'] += os.path.getsize(pathfile)
                addsuffx('.min.css', optioncss, cfile, paths, pathfile)
                compressed.append(pathfile)


        if compjs == True:
            if regjs.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est un JS !'
                weigths['js']['before'] += os.path.getsize(pathfile)
                addsuffx('.min.js', optionjs, cfile, paths, pathfile)
                compressed.append(pathfile)

        if compsvg == True:
            if regsvg.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est un SVG !'
                weigths['svg']['before'] += os.path.getsize(pathfile)
                addsuffx('.min.svg', nooption, cfile, paths, pathfile)
                compressed.append(pathfile)


        if comppng == True:
            if regpng.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est un PNG !'
                weigths['png']['before'] += os.path.getsize(pathfile)
                option = optionpng + pathfile
                compressed.append(pathfile)
                os.system('pngquant'+option)
                weigths['png']['after'] += os.path.getsize(pathfile)
                if compwebp == True:
                    print 'Compression du PNG en WEBP !'
                    sizea = os.path.getsize(pathfile)
                    weigths['webp']['before'] += sizea
                    option = optionwebp + pathfile + ' -o ' + pathfile+'.webp'
                    compressed.append(pathfile+'.webp')
                    os.system('cwebp'+option)
                    sizeb = os.path.getsize(pathfile+'.webp')
                    if(sizeb > sizea) :
                        print 'Le WEBP est plus GROS, on l\'efface -------------------------------------'
                        os.system('rm ' + pathfile+'.webp')
                    else:
                        weigths['webp']['after'] += os.path.getsize(pathfile+'.webp')



        if compjpg2wp == True:
            if regjpeg.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est un JPEG QUI VA PASSER EN WEBP !'
                weigths['jpegwebp']['before'] += os.path.getsize(pathfile)
                option = optionwebp + pathfile + ' -o ' + pathfile+'.webp'
                compressed.append(pathfile)
                os.system('cwebp'+option)
                compressed.append(pathfile+'.webp')
                weigths['jpegwebp']['after'] += os.path.getsize(pathfile+'.webp')




        if compjson == True:
            if regjson.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est un JSON !'
                weigths['json']['before'] += os.path.getsize(pathfile)
                addsuffx('.min.json', nooption, cfile, paths, pathfile)
                compressed.append(pathfile)



        if compfont == True:
            if regfont.search(cfile) is not None and pathfile not in compressed:
                print 'C\'est une FONTE !'
                suff = os.path.splitext(cfile)[1][1:]
                weigths[suff]['before'] += os.path.getsize(pathfile)
                addsuffx('.min.'+suff, nooption, cfile, paths, pathfile)
                compressed.append(pathfile)



print str(weigths)


fc = open('./compressed.dct','w');
fc.write(str(compressed))
fc.close()

for w in weigths.iteritems():
    print '------------------------------------'
    print ' ' + w[0]
    bf = w[1]['before']
    af = w[1]['after']
    if bf > 0 :
        prct = (af*100.0)/bf
        print '  minification :'
        print '   {0:.3g}% de la taille originelle'.format(prct)
        print '   GAIN : {0:.3g}%'.format(100-prct)

        if 'gz' in w[1]:
            af = w[1]['gz']
            prct = (af*100.0)/bf
            print '  gzip :'
            print '   {0:.3g}% de la taille originelle'.format(prct)
            print '   GAIN : {0:.3g}%'.format(100-prct)

        if 'br' in w[1]:
            af = w[1]['br']
            prct = (af*100.0)/bf
            print '  brotli :'
            print '   {0:.3g}% de la taille originelle'.format(prct)
            print '   GAIN : {0:.3g}%'.format(100-prct)

    else:
        print '  Pas de recompression !'
    print '------------------------------------'

````

#### htaccess pour les images

````ht
Options -MultiViews
	

<IfModule mod_rewrite.c>

	RewriteEngine on
	#RewriteBase /rmcfifa/_img/
	
	# webp
	RewriteCond "%{HTTP:Accept}"	 "\bimage/webp\b"
	RewriteCond %{REQUEST_FILENAME}.webp	-f
	RewriteRule ^(.*)\.png$            	"$1\.png\.webp" [L,NC]
	
	RewriteCond "%{HTTP:Accept}"	 "\bimage/webp\b"
	RewriteCond %{REQUEST_FILENAME}.webp	-f
	RewriteRule ^(.*)\.jpg$            	"$1\.jpg\.webp" [L,NC]
	


</IfModule>


<IfModule mod_headers.c>

		Header unset ETag
		FileETag None
		
		#Header unset Accept-Ranges
				
		# si mod_expires bug sur OVH du Cache-Control
		# Header set Cache-Control "max-age=216000, private" # 2.5 jours
		# Header set Cache-Control "max-age=604800, public" # 7 jours
		Header set Cache-Control: "public, max-age=31536000, immutable"
		
		Header unset Content-Length

		Header set content-length: totalbytes

		Header set vary: Accept-Encoding

</IfModule>



<ifModule mod_expires.c>
	# pagespeed insights n'a pas l'air de bien comprendre
	ExpiresActive On 
	#ExpiresDefault A259200
	ExpiresDefault "access plus 2592000 seconds"
	# ExpiresByType text/javascript "access plus 100 seconds"

</ifModule>
````

### Htaccess pour les js (.min.js, .js.gz, .js.br)

````ht
Options -MultiViews
	

<IfModule mod_rewrite.c>

	RewriteEngine on
	#RewriteBase /test_gz/_js/
	
	# js br
	RewriteCond "%{HTTP:Accept-encoding}"	 br
	RewriteCond %{REQUEST_FILENAME}.br	-f
	RewriteRule ^(.*)\.js$            	"$1\.js\.br" [L,NC]
	
	# js gz
	RewriteCond "%{HTTP:Accept-encoding}" "\b(x-)?gzip\b"
		#ReWriteCond %{REQUEST_FILENAME} !.+\.gz$ 
	RewriteCond %{REQUEST_FILENAME}.gz -f
	RewriteRule "^(.*)\.js"              "$1\.js\.gz" [L,NC]
	
	# js min	
	RewriteCond "%{HTTP:Accept-encoding}" !"\b(x-)?gzip\b"
	# rewrite .js -> min.js
	RewriteRule ^([^.]+)\.(js)$			$1\.min\.$2 
	# rewrite si min.js inexistant .min.js -> .js
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^([^.]+)\.(min.js)$			"$1.js" [L,NC]


</IfModule>


<IfModule mod_headers.c>

		Header unset ETag
		FileETag None
		
		#Header unset Accept-Ranges
				
		# si mod_expires bug sur OVH du Cache-Control
		# Header set Cache-Control "max-age=216000, private" # 2.5 jours
		# Header set Cache-Control "max-age=604800, public" # 7 jours
		Header set Cache-Control: "public, max-age=2592000"
		
		Header unset Content-Length

		Header set content-length: totalbytes
		Header set content-type: "text/javascript; charset=utf-8"

		Header set vary: Accept-Encoding
		
		
	
	<FilesMatch "(\.js\.gz)$">
		SetEnv no-gzip 1
		Header set content-encoding gzip
    </FilesMatch>
	
	<FilesMatch "(\.js\.br)$">
		SetEnv no-gzip 1		
		Header set content-encoding br
    </FilesMatch>

	
</IfModule>



<ifModule mod_expires.c>
	# pagespeed insights n'a pas l'air de bien comprendre
	ExpiresActive On 
	#ExpiresDefault A259200
	ExpiresDefault "access plus 2592000 seconds"
	ExpiresByType text/javascript "access plus 2592000 seconds"

</ifModule>
````
