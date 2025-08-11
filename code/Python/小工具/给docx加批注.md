inux系统上似乎没有开源项目能为docx增加批注。不过没关系，我有办法：

`.docx`文件遵从开源的“Office Open XML标准”，这意味着我们能用python的文本操作对它进行操作（实际上PPT和Excel也是）。

[OOXML标准的中文翻译](https://hellowac.github.io/ecma-376-zh-cn/)

根据OOXML标准，用以下代码就可以增加批注啦：

```python
from datetime import datetime
from typing import List
from xml.etree.ElementTree import Element, tostring

from docx import Document
from docx.opc.part import Part
from docx.opc.constants import RELATIONSHIP_TYPE, CONTENT_TYPE
from docx.opc.oxml import parse_xml
from docx.opc.packuri import PackURI
from docx.oxml import OxmlElement
from docx.oxml.ns import qn

# 命名空间
_COMMENTS_PART_DEFAULT_XML_BYTES = (
    b"""
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\r
<w:comments
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:o="urn:schemas-microsoft-com:office:office"
    xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships"
    xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math"
    xmlns:v="urn:schemas-microsoft-com:vml"
    xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing"
    xmlns:w10="urn:schemas-microsoft-com:office:word"
    xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main"
    xmlns:wne="http://schemas.microsoft.com/office/word/2006/wordml"
    xmlns:sl="http://schemas.openxmlformats.org/schemaLibrary/2006/main"
    xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main"
    xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture"
    xmlns:c="http://schemas.openxmlformats.org/drawingml/2006/chart"
    xmlns:lc="http://schemas.openxmlformats.org/drawingml/2006/lockedCanvas"
    xmlns:dgm="http://schemas.openxmlformats.org/drawingml/2006/diagram"
    xmlns:wps="http://schemas.microsoft.com/office/word/2010/wordprocessingShape"
    xmlns:wpg="http://schemas.microsoft.com/office/word/2010/wordprocessingGroup"
    xmlns:w14="http://schemas.microsoft.com/office/word/2010/wordml"
    xmlns:w15="http://schemas.microsoft.com/office/word/2012/wordml"
    xmlns:w16="http://schemas.microsoft.com/office/word/2018/wordml"
    xmlns:w16cex="http://schemas.microsoft.com/office/word/2018/wordml/cex"
    xmlns:w16cid="http://schemas.microsoft.com/office/word/2016/wordml/cid"
    xmlns:cr="http://schemas.microsoft.com/office/comments/2020/reactions">
</w:comments>
"""
).strip()


def add_comment_to_elements_in_place(
    docx_doc: Document, 
    elements: List[Element], 
    comment_text: str, 
    author: str="biubiu",
) -> None:
    '''给段落添加批注'''
    if not elements:
        return
        
	# 尝试获取已有的批注，如果不存在的话就新建一个comments.xml
    try:
        comments_part = docx_doc.part.part_related_by(
            RELATIONSHIP_TYPE.COMMENTS
        )
    except KeyError:
        comments_part = Part(
            partname=PackURI("/word/comments.xml"),
            content_type=CONTENT_TYPE.WML_COMMENTS,
            blob=_COMMENTS_PART_DEFAULT_XML_BYTES,
            package=docx_doc.part.package,
        )
        docx_doc.part.relate_to(comments_part, RELATIONSHIP_TYPE.COMMENTS)

    comments_xml = parse_xml(comments_part.blob)
    
    # 创建批注信息（id/author/date）
    comment_id = str(len(comments_xml.findall(qn("w:comment"))))
    comment_element = OxmlElement("w:comment")
    comment_element.set(qn("w:id"), comment_id)
    comment_element.set(qn("w:author"), author)
    comment_element.set(qn("w:date"), datetime.now().isoformat())

    # 创建批注文本元素
    comment_paragraph = OxmlElement("w:p")
    comment_run = OxmlElement("w:r")
    comment_text_element = OxmlElement("w:t")
    comment_text_element.text = comment_text
    comment_run.append(comment_text_element)
    comment_paragraph.append(comment_run)
    comment_element.append(comment_paragraph)

    comments_xml.append(comment_element)
    comments_part._blob = tostring(comments_xml)

    # 创建批注范围开始和结束元素，分别放在开头和结尾
    comment_range_start = OxmlElement("w:commentRangeStart")
    comment_range_start.set(qn("w:id"), comment_id)
    comment_range_end = OxmlElement("w:commentRangeEnd")
    comment_range_end.set(qn("w:id"), comment_id)
    elements[0].insert(0, comment_range_start)
    elements[-1].append(comment_range_end)

    # 为每个元素添加批注引用
    comment_reference = OxmlElement("w:r")
    comment_reference_run = OxmlElement("w:r")
    comment_reference_run_properties = OxmlElement("w:rPr")
    comment_reference_run_properties.append(
        OxmlElement("w:rStyle", {qn("w:val"): "CommentReference"})
    )
    comment_reference_run.append(comment_reference_run_properties)
    comment_reference_element = OxmlElement("w:commentReference")
    comment_reference_element.set(qn("w:id"), comment_id)
    comment_reference_run.append(comment_reference_element)
    comment_reference.append(comment_reference_run)

    elements[0].append(comment_reference)
```