---
layout: page
title: DALL-E Generated Images
permalink: /dall-e-images/
---

## DALL-E Generated Images

<div id="dall-e-images"></div>

<!-- Modal Structure -->
<div id="imageModal" class="modal">
  <span class="close">&times;</span>
  <img class="modal-content" id="modalImage">
  <div id="caption"></div>
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
    // Fetch the image URLs from the Jekyll collection
    const images = [
        {% assign images = site.static_files | where_exp: 'item', 'item.path contains "assets/img/dall-e"' %}
        {% for image in images %}
            "{{ image.path }}"{% if forloop.last == false %},{% endif %}
        {% endfor %}
    ];

    const container = document.getElementById('dall-e-images');
    shuffleArray(images).forEach(url => {
        const img = document.createElement('img');
        img.src = url;
        img.classList.add('dalle-image');
        img.style.cursor = 'pointer';
        img.onclick = function() {
            showModal(url);
        };
        container.appendChild(img);
    });

    // Function to shuffle array
    function shuffleArray(array) {
        for (let i = array.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [array[i], array[j]] = [array[j], array[i]];
        }
        return array;
    }

    // Get the modal
    var modal = document.getElementById('imageModal');
    var modalImg = document.getElementById('modalImage');
    var captionText = document.getElementById('caption');

    // Function to show modal
    function showModal(url) {
        modal.style.display = "flex";
        modalImg.src = url;
        captionText.innerHTML = url;
    }

    // Get the <span> element that closes the modal
    var span = document.getElementsByClassName("close")[0];

    // When the user clicks on <span> (x), close the modal
    span.onclick = function() {
        modal.style.display = "none";
    }

    // When the user clicks anywhere outside of the modal image, close it
    window.onclick = function(event) {
        if (event.target == modal) {
            modal.style.display = "none";
        }
    }
});
</script>

<style>
/* Style the images */
.dalle-image {
    width: 200px;
    height: auto;
    margin: 10px;
    transition: transform 0.2s;
}

.dalle-image:hover {
    transform: scale(1.1);
}

/* The Modal (background) */
.modal {
    display: none; /* Initially hide the modal */
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0,0,0,0.9); /* Apply background color only when visible */
    justify-content: center;
    align-items: center;
}

/* Modal Content (image) */
.modal-content {
    margin: auto;
    display: block;
    max-width: 80%;
    max-height: 80%;
}

/* Caption of Modal Image */
#caption {
    margin: auto;
    display: block;
    width: 80%;
    max-width: 700px;
    text-align: center;
    color: #ccc;
    padding: 10px 0;
    height: auto;
}

/* Add Animation - Zoom in the Modal */
.modal-content, #caption {
    -webkit-animation-name: zoom;
    -webkit-animation-duration: 0.6s;
    animation-name: zoom;
    animation-duration: 0.6s;
}

@-webkit-keyframes zoom {
    from { -webkit-transform: scale(0) }
    to { -webkit-transform: scale(1) }
}

@keyframes zoom {
    from { transform: scale(0) }
    to { transform: scale(1) }
}

/* The Close Button */
.close {
    position: absolute;
    top: 15px;
    right: 35px;
    color: #f1f1f1;
    font-size: 40px;
    font-weight: bold;
    transition: 0.3s;
}

.close:hover,
.close:focus {
    color: #bbb;
    text-decoration: none;
    cursor: pointer;
}
</style>
