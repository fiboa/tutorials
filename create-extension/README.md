# How to create a fiboa extension

In this tutorial we'll create a new fiboa extension from scratch and 
make use of it in a dataset.

The list of existing extensions:
<https://fiboa.github.io/extensions>

A list of proposed extensions:
<https://github.com/fiboa/extensions/issues>  
Feel free to add your ideas or take over specifying any of them!

German crop types:
- hbn (string, enum)
- hbn_text (string)
- ground_truth (boolean)

1. We need to create a new repository.
   There is a detailed instruction here:
   <https://github.com/fiboa/extensions/blob/main/README.md#adding-a-new-extension>
2. We need to clone the new reporistory (e.g. `git clone https://github.com/username/x-extension`) and open it (`cd x-extension`).
2. We use fiboa CLI to rename the placeholders in the template.