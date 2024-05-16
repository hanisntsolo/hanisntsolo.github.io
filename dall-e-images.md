---
layout: page
title: DALL-E [OpenAI]
permalink: /dall-e-images/
cover-img:
  - "/assets/img/dall-e/DALL·E 2024-05-16 21.11.39 - A group photo at an Indian wedding with the couple sitting on a decorated stage. The groom is wearing a white sherwani with a turban, and the bride is wearing lehanga.webp"
  - "/assets/img/dall-e/DALL·E 2024-05-16 17.06.48 - A photo-realistic scene of Pune during the rains, with Marathi people enjoying the weather. The streets are wet, with reflections of the city lights..webp"
  - "/assets/img/dall-e/DALL·E 2024-05-17 01.27.32 - A cinematic and dramatic scene with two powerful wizards, reminiscent of Voldemort and Dumbledore, engaged in an intense duel. Voldemort appears more .webp"
  - "/assets/img/dall-e/DALL·E 2024-05-17 01.27.54 - A photorealistic image of a character resembling Dumbledore, showcasing his magical capabilities..webp"
---

## DALL-E Generated Images

<div id="dall-e-images"></div>

<!-- Modal Structure -->
<div id="imageModal" class="modal">
  <span class="close">&times;</span>
  <img class="modal-content" id="modalImage">
  <div id="caption"></div>
  <a class="prev">&#10094;</a>
  <a class="next">&#10095;</a>
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
            currentIndex = images.indexOf(url);
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

    let currentIndex = 0;

    // Get the modal
    const modal = document.getElementById('imageModal');
    const modalImg = document.getElementById('modalImage');
    const captionText = document.getElementById('caption');
    const prev = document.querySelector('.prev');
    const next = document.querySelector('.next');

    // Function to show modal
    function showModal(url) {
        modal.style.display = "flex";
        modalImg.src = url;
        captionText.innerHTML = url;
    }

    // Get the <span> element that closes the modal
    const span = document.getElementsByClassName("close")[0];

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

    // Navigate to the previous image
    prev.onclick = function() {
        currentIndex = (currentIndex === 0) ? images.length - 1 : currentIndex - 1;
        showModal(images[currentIndex]);
    }

    // Navigate to the next image
    next.onclick = function() {
        currentIndex = (currentIndex === images.length - 1) ? 0 : currentIndex + 1;
        showModal(images[currentIndex]);
    }

    // Handle swipe events for touch screens
    let startX;
    modalImg.addEventListener('touchstart', function(event) {
        startX = event.touches[0].clientX;
    });

    modalImg.addEventListener('touchend', function(event) {
        const endX = event.changedTouches[0].clientX;
        if (startX > endX + 50) {
            next.onclick();
        } else if (startX < endX - 50) {
            prev.onclick();
        }
    });
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
    position: absolute;
    bottom: 20px;
    width: 100%;
    text-align: center;
    color: #ccc;
    font-size: 0.9em;
    font-style: italic;
    padding: 10px;
    box-sizing: border-box;
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

/* Navigation arrows */
.prev, .next {
    cursor: pointer;
    position: absolute;
    top: 50%;
    width: auto;
    margin-top: -22px;
    padding: 16px;
    color: white;
    font-weight: bold;
    font-size: 24px;
    transition: 0.3s;
    user-select: none;
}

.prev:hover, .next:hover {
    background-color: rgba(0,0,0,0.8);
}

.prev {
    left: 0;
    border-radius: 0 3px 3px 0;
}

.next {
    right: 0;
    border-radius: 3px 0 0 3px;
}
</style>
