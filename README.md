let s = function (p) {
    let movingPoint;
    let pointA, pointB;
    let pointSpacing = 200;

    p.setup = function () {
        p.createCanvas(p.windowWidth, p.windowHeight);
        p.textFont('Georgia');

        let centerX = p.width / 2;
        let centerY = p.height / 2;

        pointA = p.createVector(centerX - pointSpacing, centerY);
        pointB = p.createVector(centerX + pointSpacing, centerY);

        movingPoint = p.createVector(centerX, centerY - 150);
    };

    p.draw = function () {
        p.background(255);

        // Draw parabola
        drawParabola();

        // Line between points A and B
        p.stroke(0);
        p.strokeWeight(2);
        p.line(pointA.x, pointA.y, pointB.x, pointB.y);

        // Draw points A and B
        p.strokeWeight(10);
        p.point(pointA.x, pointA.y);
        p.point(pointB.x, pointB.y);

        // Draw moving point
        p.stroke(255, 0, 0);
        p.point(movingPoint.x, movingPoint.y);

        // Projection and perpendicular
        let AP = p5.Vector.sub(movingPoint, pointA);
        let AB = p5.Vector.sub(pointB, pointA);
        let AB_unit = AB.copy().normalize();
        let projectionLength = AP.dot(AB_unit);
        let projectionPoint = p5.Vector.add(pointA, AB_unit.copy().mult(projectionLength));

        p.stroke(0, 255, 0);
        p.strokeWeight(2);
        p.line(movingPoint.x, movingPoint.y, projectionPoint.x, projectionPoint.y);

        // Distance and side info
        let distance = p5.Vector.dist(movingPoint, projectionPoint);
        let side = (AB.x * AP.y - AB.y * AP.x) > 0 ? "Left" : "Right";

        p.noStroke();
        p.fill(0);
        p.textSize(16);
        p.text(`Projection: (${projectionPoint.x.toFixed(1)}, ${projectionPoint.y.toFixed(1)})`, 20, 20);
        p.text(`Distance from line: ${distance.toFixed(2)}`, 20, 40);
        p.text(`Side: ${side}`, 20, 60);

        // Draw perpendicular boxplot and ridge
        drawPerpBoxAndRidge(projectionPoint, AB_unit, 100);
    };

    function drawParabola() {
        p.stroke(100, 100, 255);
        p.noFill();
        p.beginShape();
        let centerX = p.width / 2;
        let centerY = p.height / 2 + 150;
        let scaleX = 0.01; // controls width
        let scaleY = 0.005; // controls height
        for (let x = -p.width / 2; x <= p.width / 2; x += 2) {
            let y = scaleY * (x * x);
            p.vertex(centerX + x, centerY - y);
        }
        p.endShape();
    }

    function drawPerpBoxAndRidge(basePoint, AB_unit, length) {
        let perpUnit = p.createVector(-AB_unit.y, AB_unit.x);
        let perpPoints = [];

        let numPoints = 100;
        for (let i = 0; i < numPoints; i++) {
            let offset = p.randomGaussian() * 20; // spread
            let pt = p5.Vector.add(basePoint, perpUnit.copy().mult(offset));
            perpPoints.push(pt);
        }

        // Project offsets
        let offsets = perpPoints.map(pt => p5.Vector.sub(pt, basePoint).dot(perpUnit));
        offsets.sort((a, b) => a - b);

        // Boxplot stats
        let q1 = p.lerp(offsets[0], offsets[Math.floor(numPoints / 2)], 0.5);
        let median = offsets[Math.floor(numPoints / 2)];
        let q3 = p.lerp(offsets[Math.floor(numPoints / 2)], offsets[numPoints - 1], 0.5);

        let minVal = offsets[0];
        let maxVal = offsets[numPoints - 1];

        // Draw boxplot
        p.stroke(0);
        p.strokeWeight(1);
        p.noFill();
        let q1Pt = p5.Vector.add(basePoint, perpUnit.copy().mult(q1));
        let q3Pt = p5.Vector.add(basePoint, perpUnit.copy().mult(q3));
        p.line(q1Pt.x, q1Pt.y, q3Pt.x, q3Pt.y); // box
        p.point(q1Pt.x, q1Pt.y);
        p.point(q3Pt.x, q3Pt.y);

        // Median
        let medianPt = p5.Vector.add(basePoint, perpUnit.copy().mult(median));
        p.stroke(255, 0, 0);
        p.line(medianPt.x - 5, medianPt.y - 5, medianPt.x + 5, medianPt.y + 5);
        p.line(medianPt.x - 5, medianPt.y + 5, medianPt.x + 5, medianPt.y - 5);

        // Whiskers
        let minPt = p5.Vector.add(basePoint, perpUnit.copy().mult(minVal));
        let maxPt = p5.Vector.add(basePoint, perpUnit.copy().mult(maxVal));
        p.stroke(0);
        p.line(minPt.x, minPt.y, q1Pt.x, q1Pt.y);
        p.line(q3Pt.x, q3Pt.y, maxPt.x, maxPt.y);

        // Ridge plot
        p.noFill();
        p.beginShape();
        for (let t = -length; t <= length; t += 2) {
            let density = offsets.filter(v => p.abs(v - t) < 10).length;
            let height = density * 2;
            let ridgePt = p5.Vector.add(basePoint, perpUnit.copy().mult(t));
            p.vertex(ridgePt.x, ridgePt.y - height);
        }
        p.endShape();
    }

    p.mouseDragged = function () {
        movingPoint.x = p.mouseX;
        movingPoint.y = p.mouseY;
    };

    p.windowResized = function () {
        p.resizeCanvas(p.windowWidth, p.windowHeight);
    };
};

new p5(s);
