# My notes

## Package pillow
resize image files 
    
    pip install Pillow

No routes.py importar

    from PIL import Image

> Vamos redimensionar antes de salvar
    output_size = (125, 125)
    i = Image.open(form_picture)
    i.thumbnail(output_size)
    i.save(picture_path)